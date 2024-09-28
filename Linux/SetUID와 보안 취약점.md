```toc
```
# 리눅스의 파일 권한 체계

![[Pasted image 20240924180925.png]]
(출처: https://medium.com/@techbytebykiru/understanding-linux-file-permissions-a-comprehensive-guide-b5f3df1b0c5a)

리눅스에서는 파일에 대한 권한 체계가 위와 같이 이루어져있다. `onwer - group - other users`에 따라 3비트로 나누어 권한을 구분하고 이를 `rwx(421)`로 표현한다. 

- `owner`: 파일 소유주의 권한
- `group`: 그룹 구성원들에 대한 권한
- `other users`: 그 외 모든 사용자에 대한 권한

 `rwx`는 각각 `read`, `write`, `execute`로 읽기, 쓰기, 실행 권한이다. 만약 읽기와 실행 권한만 있다면 `r-x`로 표현하고 값은 5가 되는 식이다. 그래서 파일의 권한을 수정하는 `chmod` 명령어의 경우 `chmod 644 [file]`와 같이 사용을 할 수 있는 것이고 `644`로 권한을 부여한 경우 `rw-r--r--`가 되는 것이다

![[Pasted image 20240924180714.png]]

우리가 흔히 `ls -al`을 했을 때 볼 수 있는 화면이다. 권한 뒤에 `@` 와 `+`가 붙어 있는 애들이 있는데 이건 각각 `확장 속성`과 `ACL`이다. 

- `확장 속성`: 파일이나 디렉토리에 추가적인 메타 데이터를 저장하는 기능.
- `ACL`: 파일이나 디렉토리에 더 세밀한 권한 설정을 할 수 있는 기능.

이 글의 범위가 넘어가는 부분이기 때문에 자세한 설명을 생략하겠다. 하지만 유닉스 파일 시스템에 반드시 필요하고 유용한 기능들이므로 자세히 찾아보길 추천한다.

# SetUID

앞서 설명한 권한 체계는 리눅스에 높은 보안성을 가져다 주지만, 반대로 사용자의 불편과 생산성 저하를 가져올 수 있다. 예를 들어 강력한 보안이 필요한 파일이 있다고 가정해보자. 이를테면 유저들의 패스워드가 저장되는 `/etc/shadow` 파일과 같이 말이다.

해당 파일은 가장 민감한 패스워드를 가지고 있는 파일인 만큼 당연하게도 `root`가 파일의 소유주여야 할 것이고, 다른 유저들은 이 파일에 대한 읽기/쓰기가 모두 불가능 해야 할 것이다.

하지만 그렇다고 오직 `root`만이 각 유저들의 패스워드를 변경할 수 있다면 너무나도 불편할 것이다. `root` 접속 권한이 없는 사람은 자기 계정의 패스워드를 변경 할 수 없거나, 변경이 필요할 때 마다 `root` 접속 권한이 있는 사람에게 요청을 해야할 것이다.

이와 같은 케이스를 위해 **리눅스에서는 파일을 실행할 때, 파일 소유주의 권한으로 실행할 수 있는 기능이 있는데, 그게 바로 `SetUID`** 이다. 이 `SetUID`를 이용하는 명령어는 대표적으로 `passwd` 명령어가 있다.

![[TIL/Linux/attachments/Pasted image 20240815165732.png]]

`passwd`의 권한을 보면 **`rwx`가 아니라 `rws`라고 표현**되어 있는 것을 볼 수 있다. `SetUID`가 적용되어 있는 파일의 경우는 권한이 `rws`로 표현이 되고 비트로는 앞에 4를 붙여 표현한다. 즉, `rws`는 `47`이 되고 위 `passwd`의 경우에는 `4755`가 되는 것이다. `chmod`를 사용할 때도 `chmod u+s [file]`와 같이 사용하거나 `chmod 4755 [file]`와 같이 사용하면 된다.

![[TIL/Linux/attachments/Pasted image 20240815165703.png]]

앞서 설명했다 시피 유저들의 패스워드가 저장되는 `/etc/shadow` 파일의 경우 `root`만이 읽기/쓰기가 가능하다. 하지만 **`passwd`에 `SetUID`가 설정이 되어있고, `passwd`의 소유주는 `root`이다. 그렇기에 일반 유저가 `passwd`를 실행 할 때 권한이 상승되어 `root`의 권한으로 명령어를 실행하게 되고 일반 유저도 `/etc/shadow` 파일을 수정하여 패스워드를 변경할 수 있게 되는 것이다.**

다른 예로 하드웨어 장치에 접근해야 할 경우가 있다. 예를 들어 `ping`과 같은 명령어가 있다. `ping`을 사용하기 위해선 네트워크 장치에 대한 접근 권한이 필요한데, 이와 같이 하드웨어 장치에 직접적으로 접근하는 것 또한 보안상 예민한 부분이기에 권한이 제한적이기 마련이다. 그렇기에 `ping` 또한 `SetUID`를 통해 실행 시 `root`의 권한을 잠시 빌려와 네트워크 장치에 접근을 할 수 있다.

# SetUID의 보안 취약점 예시

`SetUID`의 권한 상승은 사용자의 편의성을 위해 존재해야 하지만, 권한 상승이라는 자유로움으로 인해 보안적인 허점을 만들기도 한다. 그러한 허점의 예시로 CTF 문제 중 하나를 들겠다. 만약에 **내가 필요한 권한을 가진 유저가 소유주이고 `SetUID`가 적용된 파일이 있다고 가정**을 해보자. 예를 들면 다음 파일과 같이 말이다.

![[TIL/Linux/attachments/Pasted image 20240823184044.png]]

이 예시는 `42 Seoul`의 outer circle 과정 중, CTF 문제를 다루는 과제인  `snow-crash`의 문제 중 하나이다. 보다시피 `Lua` 스크립트가 하나 존재하고 **소유주는 `flag11`이며 `SetUID`가 적용**되어 있는 것을 확인 할 수 있다.

해당 과제는 각 레벨마다 `flag레벨` 유저로 로그인을 하여 `getflag` 명령어를 실행 시켜 `flag`를 얻어야 하는 CTF 문제 풀이 과제로, `level11` 유저로 로그인을 한 뒤 `flag11`으로 로그인을 할 방법을 찾아야 하는 상태이다. 그래야만 `getflag`로 다음 레벨로 넘어 갈 수 있는 `flag`를 획득 할 수 있기 때문이다.

다만, 이번 문제의 경우엔 `SetUID`를 활용하는 문제로 **`flag11`으로 로그인을 하지 않아도 된다.**  앞서 설명했다시피, `SetUID`는 파일의 소유주 권한으로 실행이 되기 때문에 `level11`에 `Command Injection`을 통해 `getflag`를 실행 시키게 하면, `flag11`으로 로그인을 하는 것과 똑같은 결과를 얻을 수 있다!

```lua
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```

해당 `Lua` 스크립트의 내용은 위와 같다. `127.0.0.1`의 `5151`포트를 사용하고 있으며, 클라이언트에게 전송받은 데이터를 `hash()`라는 함수로 해싱을 한 뒤 그 값을 다른 해싱값인 `f05d1d066fb246efe0c6f7d095f909a7a0cf34a0`와 비교를 하는 간단한 프로그램이다.

이 프로그램의 보안 취약점은 `io.popen()`에 있다. 이는 시스템의 명령어를 실행한 뒤 그 결과를 반환 받는 함수로, `io.popen("echo "..pass.." | sha1sum", "r")`을 보면 클라이언트의 입력 값을 시스템의 `sha1sum` 명령어에게 전달하여 해싱을 수행하는 것을 볼 수 있다.

언뜻 보았을 때는 시스템의 `sha1sum`을 이용하기에 따로 라이브러리나 복잡한 구현이 필요 없어 좋은 방법이라고 볼 수 있지만, **시스템 명령어를 호출하는데 그 인자로 사용자의 입력 값을 그대로 사용하고 있다. 이는 `Command Injection`에 전면으로 노출되는 가장 취약한 방법**이며, 그를 방어하는 어떠한 코드도 존재하지 않는다. 더군다나 앞서 보았듯이 이 스크립트는 `SetUID`까지 설정이 되어 있으니 권한 탈취 마저도 가능한 상태이다.

물론 현실에서 이렇게 대문을 열어재낀 서버는 존재하지 않을 것이다. 어디까지나 CTF 문제 중 하나이며, 우리는 공부를 목적으로 어떻게 하면 이 녀석을 활용하여 `flag`를 얻을 지에 집중하면 된다.

![[TIL/Linux/attachments/Pasted image 20240823190616.png]]

`nc` 명령어를 통해 접속을 하면 스크립트 코드 대로 `Password: `라는 문구가 나오며 패스워드를 입력하라고 한다. `SetUID`를 통해 `flag11`의 권한으로 실행이 되고 있으므로, 우리는 `getflag`를 통해 간단히 `flag`를 얻을 수 있다. `io.popen()` 때문에 그 결과를 표준 출력으로 얻을 수는 없기에, 그 결과를 어딘가에 리다이렉션 해주기만 하면 된다. 그 후 결과를 리다이렉션한 `/tmp/tmp`를 출력해보면 `flag`를 성공적으로 탈취한 것을 확인할 수 있다!

위 예시는 CTF 문제라 보안 취약점이 와닿지 않을 수 있지만, [CVE-2018-14665](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-14665)와 같이 실제 공격 사례 또한 존재한다고 한다. 유저의 권한을 일시적으로 변경하는 기능인 만큼 보안적인 허점을 만들 수 있기에 사용에 있어서 각별히 주의가 필요할 것이다.
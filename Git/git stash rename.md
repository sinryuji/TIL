# 0. stash 할 때 이름 지정 방법
```shell
git stash save "save로 이름 지정"
```

# 1. 명령어 alias 설정
```shell
git config --global alias.stash-rename '!_() { rev=$(git rev-parse $1) && git stash drop $1 || exit 1 ; git stash store -m "$2" $rev; }; _'
```

# 2. stash-rename으로 이름 바꾸기
```shell
git stash-rename stash@{[바꿀 Stash 번호]} [바꿀 Stash name]
```

예시
```shell
git stash-rename stash@{0} "수업 디자인 목록 WIP"
```


# 출처
https://jw910911.tistory.com/118
# CI/CD란?

`CI/CD`란 `지속적 통합(Continuous Integration)`과 `지속적 배포(Continuous Deployment)`의 약자로서, 소프트웨어 개발 프로세스를 혁신적으로 개선하는데 중요한 역할을 수행한다. `CI/CD`를 통해 코드 변경 사항을 자동으로 빌드, 테스트 및 배포함으로써 개발 속도를 높이고 안정성을 확보할 수 있다. 이러한 자동화 방번론은 개발자가 수동으로 빌드와 배포 과정을 반복하지 않아도 되게 하여 많은 시간을 절약할 수 있다.

# Workflow

프로젝트를 통합하고 배포하는데 있어서 일련의 프로세스가 있을 것이다. `Github Actions`에서는 이런 프로세스를 `Workflow`라 칭하며 이를 `yml` 파일을 통해 자동화할 수 있도록 해준다.

```yml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.
# This workflow will build a Java project with Gradle and cache/restore any dependencies to improve the workflow execution time
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-java-with-gradle

name: Java CI with Gradle

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@bd5760595778326ba7f1441bcf7e88b49de61a25 # v2.6.0
      with:
        arguments: build

```

프로젝트에서 `Actions` 탭을 누르면 기본적으로 Suggestion 해주는 워크플로우가 몇가지 있는데 그 중 `Java with Gradle`이라는 워크플로우다. 이 파일을 통해 Github Actions의 워크플로우가 어떻게 구성 되는지 알아보자.

### Event

워크플로우를 실행하는 시점이다. `push`와 `pull_request`는 이름에서도 알 수 있다시피 커밋이 push 되거나 pull request가 생성 되었을 때 워크플로우를 실행 할 수 있고, `schedule`을 통해 일정 주기마다 워크플로우를 실행 할 수 있다.
뿐만 아니라 `reposiotry_dispatch`를 통해 무려 다른 레포지토리의 워크플로우를 실행 할 수도 있다!

### Runner

워크플로우를 실행하는 서버이다. Github에서는 `Ubuntu`, `Windows`, `macOS` 환경의 러너를 제공한다. 그 외에 특정 OS나 하드웨어 스펙에서 실행하고 싶으면 `Self-Hosted Runner`를 사용할 수 있다.
### Job

워크플로우의 기본 단위이다. 이는 더 작은 단위인 `step`으로 이루어져 있다. 워크플로우는 기본적으로 이 `job`들을 병렬적으로 실행하며 순차적으로 실행하도록 설정 할 수도 있다. 예를 들어 빌드와 테스트 코드 실행의 두 작업은 순차적으로 실행할 수도 있으며 이 경우에 빌드가 실패하면 테스트 작업은 실행되지 않는다.

### Step

`step`은 쉘 스크립트 일수도 있고, 일반적인 커맨드일 수도 있다. `job`내의 `step`들은 순차적으로 실행되며, 모두 동일한 `runner`에서 실행되므로 **상호 데이터를 공유할 수 있다.**

### Action

복잡하지만 자주 사용되는 작업 단위를 재사용이 가능하도록 만든 실행 단위이다. `action`은 `GitHub Marketplace`에서 공유 할 수 있고, 자신의 워크플로우에 다른 사람의 액션을 가져와 사용할 수 있다.

# Workflow 관리

### 민감한 정보 관리
워크플로우가 비밀번호나 인증서 같은 민감한 정보를 사용한다면 Github에 secret으로 저장하여 환경 변수로 사용할 수 있다.

```yml
jobs:
  example-job:
    runs-on: unbuntu-latest
    steps:
      - name: Retrieve secret
        # 환경변수로 저장하고
        env:
          super_secret: ${{ secrets.SUPERSECRET }}
        # 저장한 환경변수를 활용한다.
        run: |
          example-command "$super_secret"
```

### 의존적인 작업 구성
기본적으로 작업은 병렬적으로 수행되지만 다른 작업이 완전히 끝난 후에 작업을 실행시키고 싶다면 `needs` 키워드를 통해 작업이 의존성을 갖도록 지정하면 된다.
```yml
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - run: ./setup_server.sh
  build:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - run: ./build_server.sh
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: ./test_server.sh
```

### 빌드 매트릭스
워크플로우가 다양한 OS, 플랫폼, 언어의 조합에서 테스트를 실행하려는 경우 빌드 매트릭스를 활용하면 된다. 빌드 옵션을 배열로 받는 `strategy` 키워드를 사용하면 된다.

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        # 다양한 버전의 Node.js를 이용하여 작업을 여러번 실행
        node: [6, 8, 10]
    steps:
      - uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node }}
```

### 종속성 캐싱
Github의 러너는 각 작업에서 새로운 환경으로 실행되므로 작업들이 종속성을 재사용하는 경우 파일들을 캐싱하여 성능을 높이면 된다. 캐시를 생서앟면 해당 저장소의 모든 워크플로우에서 사용할 수 있다.

```yml
jobs:
  example-job:
    steps:
      - name: Cache node modules
        uses: actions/cache@v2
        env:
          cache-name: cache-node-modules
        with:
          # `~/.npm` 디렉토리를 캐시해 성능을 높인다
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
```


##### 출처
https://meetup.nhncloud.com/posts/286

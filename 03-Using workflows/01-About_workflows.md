## workflow란? 

하나 이상의 job을 실행하는 구성가능한 자동화된 프로세스이다. 
repository에 check in 된 yaml 파일에서 정의되고 수동 또는 설정한 일정 또는 특정한 event가 발생할 때 실행될 수 있다. 

workflow는 `.github/workflows` 폴더에 위치해야 한다. 당연히 해당 폴더에는 여러 개의 workflow가 존재할 수 있다. 

## workflow basics 

- workflow가 갖고 있는 기본 요소
  
  1. workflow를 실행시킬 하나 이상의 `event` (ex. pull requetst, push)
  2. runner machine에서 실행해서 일련의 step 들이 동작하는 하나 이상의 `job`
  3. 각 `step`에서는 action을 정의하거나 동작시킬 수 있다. 그러한 action들이 workflow를 간단하게 만들어 준다.

![image](https://github.com/sjHong645/github-action-study/assets/64796257/fc1e9323-5a0d-4943-af13-5d562512c76e)

## Triggering a workflow 

`trigger`란 workflow를 실행하도록 하는 `event`이다. 이벤트들은 다음과 같다. 

- workflow의 repository에서 발생한 이벤트
- Github 외부에서 발생하고 Github에서 `repository_dispatch` 이벤트를 발생시키는 이벤트
- 예약된 시간
- 수동

[trigger에 대한 자세한 내용](https://docs.github.com/ko/actions/using-workflows/triggering-a-workflow)  
[workflow를 trigger하는 event](https://docs.github.com/ko/actions/using-workflows/events-that-trigger-workflows)

## 예제 

앞서 말한 것 처럼 해당 workflow 파일을 `.yml` 확장자로 설정해서 repository의 `.github/workflows` 디렉토리에 저장해야 한다. 

- 파일 : [각 구문별 해설](https://docs.github.com/en/actions/using-workflows/about-workflows#understanding-the-workflow-file)
```
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

- 파일 시각화
![image](https://github.com/sjHong645/github-action-study/assets/64796257/56b96fa7-8ef1-44b9-9059-8c269f2bca73)


## starter workflow 사용 

Github는 미리 설정된 starter workflow를 제공해서 나만의 CI(Continuous Integration) workflow를 만들 수 있다. 

- [모든 starter workflow들](https://github.com/actions/starter-workflows)

## workflow 고급 특징 

고급 특징 중 몇 가지만 간단하게 살펴보도록 하겠다. 

### 비밀값 저장 

workflow가 민감한 데이터를 사용한다면 이 값들을 Github에서 `secrets`으로 저장해서 workflow의 환경변수로 사용할 수 있다. 

ex)  
```
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
      - name: Retrieve secret
        env:
          super_secret: ${{ secrets.SUPERSECRET }}
        run: |
          example-command "$super_secret"
```

[자세한 내용](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

### dependent job 만들기 

기본적으로 workflow의 job은 동시에 병렬적으로 실행된다. 
만약, 다른 작업이 완료되고 나서 실행해야 하는 job이 있다면 `needs`라는 키워드를 이용해서 종속성을 만들 수 있다. 

ex)  
```
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - run: ./setup_server.sh
  build:
    needs: setup # setup이라는 job이 끝난 다음에 실행할 수 있는 작업 
    runs-on: ubuntu-latest
    steps:
      - run: ./build_server.sh
  test:
    needs: build # build라는 job이 끝난 다음에 실행할 수 있는 작업 
    runs-on: ubuntu-latest
    steps:
      - run: ./test_server.sh
```

### matrix 사용 

matrix를 사용하면 하나의 job에서 변수의 조합을 바탕으로 여러 개의 job을 동작시키도록 할 수 있다.

matrix는 `strategy`라는 키워드를 사용해서 만든다. strategy 키워드는 배열로 빌드 옵션을 수신한다. 

ex) 다른 버전의 Node.js를 사용해서 job을 여러 번 실행하도록 matrix를 만들었다.  
```
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [14, 16]
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
```

[자세한 내용](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

### 종속성 캐싱 

job이 정기적으로 종속성을 재사용한다면 해당 파일들을 캐싱하는 방법을 고려할 수 있다.  
한 번 캐싱해놓으면 똑같은 repository에 있는 모든 workflow에서 사용할 수 있다. 

ex) `~/.npm` 디렉토리를 캐싱하느 예시 
```
jobs:
  example-job:
    steps:
      - name: Cache node modules
        uses: actions/cache@v3
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
```

[자세한 내용](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

### DB 및 서비스 컨테이너 사용 

job에 DB 또는 cache 서비스가 필요한 경우 `service` 키워드를 사용하여 서비스를 호스팅하는 임시 컨테이너를 만들 수 있다. 
그러면 해당 job의 모든 step에서 결과값으로 나온 컨테이너를 사용할 수 있고 job이 끝나면 삭제된다. 

ex) service를 사용해서 postgres 컨테이너를 만들고 서비스를 연결할 때 `node`를 사용한 예시 

```
jobs:
  container-job:
    runs-on: ubuntu-latest
    container: node:10.18-jessie
    services:
      postgres:
        image: postgres
    steps:
      - name: Check out repository code
        uses: actions/checkout@v4
      - name: Install dependencies
        run: npm ci
      - name: Connect to PostgreSQL
        run: node client.js
        env:
          POSTGRES_HOST: postgres
          POSTGRES_PORT: 5432
```

### label을 사용한 workflow 라우팅 

특정 유형의 runner가 job에서 동작하길 원한다면 job이 어디에서 실행되는지 제어하기 위해서 label을 사용할 수 있다.  


ex) workflow에서 label을 사용해서 필요한 runner를 지정하는 예시 
```
jobs:
  example-job:
    runs-on: [self-hosted, linux, x64, gpu]
```

workflow는 `runs-on` 배열에 있는 label만 동작시킬 수 있다. 

[self-host에 대한 내용](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-labels-with-self-hosted-runners) / [Github-host에 대한 내용](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)

### workflow 재사용 

하나의 workflow가 다른 workflow를 호출할 수 있다. 이렇게 재사용함으로써 중복을 피하고 workflow를 유지하기 더욱 쉬워진다.  
[자세한 내용](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

### 환경 사용 

[자세한 내용](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)

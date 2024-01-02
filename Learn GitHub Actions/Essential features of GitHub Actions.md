## 학습 목표

1. 변수 사용, 스크립트 실행, job 간의 데이터 및 artifact 공유와 같은 필수적인 workflow 생성 방법을 안다. 

## workflow에서 변수 사용하기 

ex) `POSTGRES_HOST`와 `POSTGRES_PORT`라는 이름을 가진 변수를 만들어낸 workflow
```
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
      - name: Connect to PostgreSQL
        run: node client.js
        env:
          # 아래와 같이 설정한 변수들은 이제 node client.js 스크립트에서 사용할 수 있다. 
          POSTGRES_HOST: postgres
          POSTGRES_PORT: 5432
          
```

각 workflow run에는 기본적인 환경 변수가 포함되더 있다. 
이외에 추가적인 사용자 변수를 사용하고 싶다면 위와 같은 방식으로 변수를 설정할 수 있다. [변수에 대한 자세한 내용](https://docs.github.com/ko/actions/learn-github-actions/variables#default-environment-variables)

## workflow에서 스크립트 추가 

ex) `run 키워드`를 사용 ⇒ `npm install -g bats` 명령을 실행
```
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
      - run: npm install -g bats
```

위와 같이 `run 키워드`를 이용해서 runner에서 script를 실행할 수 있다. 

ex) job의 서로 다른 step에서 각기 다른 script 실행 
```
jobs:
  example-job:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./scripts
    steps:
      - name: Check out the repository to the runner
        uses: actions/checkout@v4  
      - name: Run a script 
        run: ./my-script.sh # script 실행 
      - name: Run another script
        run: ./my-other-script.sh # 다른 script 실행 
```

여기서 실행되는 `script의 위치`는 run 명령어에 대한 기본 작업 디렉토리를 별도로 설정해줘야 한다. [관련 내용](https://docs.github.com/ko/actions/using-jobs/setting-default-values-for-jobs)

[run 키워드에 대한 자세한 내용](https://docs.github.com/ko/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun)

## jobs 간에 data 공유 

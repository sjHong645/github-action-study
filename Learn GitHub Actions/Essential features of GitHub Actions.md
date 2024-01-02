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

- aritfact : 코드를 빌드하고 테스트할 때 생성되는 파일 ⇒ 동일한 workflow에서 하나의 job이 다른 job과 공유하고 싶은 파일을 생성하거나 나중에 참조를 위해 파일을 저장하고 싶을 때 github에서는 `artifact`라는 걸 저장한다.

run 내에서 실행되는 모든 action과 workflow는 run의 artifact에 대한 쓰기 권한을 갖는다. 

ex) 파일을 하나 생성하고 그 파일을 `artifact`로 업로드 하고 싶다.
```
jobs:
  example-job:
    name: Save output
    runs-on: ubuntu-latest
    steps:
      - shell: bash
        run: |
          expr 1 + 1 > output.log # output.log라는 파일을 하나 생성
      - name: Upload output file
        uses: actions/upload-artifact@v3 # artifact 업로드를 위해 실행되는 action 
        with:
          name: output-log-file # artifact의 이름은 output-log-file
          path: output.log # output.log 라는 파일을 경로로 설정해서 해당 파일을 artifact로 만들어서 업로드 실행 
```

ex) 이번에는 `artifact`를 다운로드 하고 싶다. 
```
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
      - name: Download a single artifact
        uses: actions/download-artifact@v3
        with:
          name: output-log-file # 이름이 output-log-file인 artifact를 다운로드 한다. 
```

만약, 동일한 workflow run에서 artifact를 다운로드 하고 싶다면 다운로드 작업에 `needs: upload-job-name`을 지정해야 한다. 
그래야 upload가 끝날 때 까지 다운로드를 실행하지 않는다. 

[자세한 내용](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

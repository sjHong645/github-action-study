### About Contexts 

컨텍스트(context) : workflow의 실행(run), 변수, runner 환경, jobs, steps에 대한 정보에 접근하는 방법 

아래 구문을 통해 context에 접근할 수 있다.
```
${{ <context> }}
```

[컨텍스트 목록](https://docs.github.com/ko/actions/learn-github-actions/contexts#about-contexts)

위 목록에 있는 context 정보를 아래 2가지 방법으로 접근할 수 있다.

- 인덱스 접근법 : github['sha']
- 속성 역참조 접근법 : github.sha

### Context를 언제 사용할 지 결정하기 

Github Action은 `default 변수`와 `context`를 담고 있다. 이 변수들은 workflow에서 서로 다른 지점에서 사용된다. 

- 기본 환경 변수(Default Environment Variable) : job을 실행하고 있는 runner에서만 존재하는 변수. [모든 변수값들](https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables)
- Contexts : 대부분의 context는 workflow의 모든 지점에서 사용할 수 있다. [Context 사용 범위](https://docs.github.com/en/actions/learn-github-actions/contexts#context-availability)

ex) 2가지 종류의 변수가 job에서 어떻게 사용되는지 보여주는 예시 
```
name: CI
on: push
jobs:
  prod-check:
    if: ${{ github.ref == 'refs/heads/main' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production server on branch $GITHUB_REF"
```
- if문 : `github.ref` 컨텍스트를 사용해서 현재 branch의 이름을 확인한다.  
- steps > run : `$GITHUB_REF` 변수를 사용했다.

### Context 사용 범위 

아래 링크에 있는 함수들은 workflow 내에서 각 context 및 특수 함수를 사용할 수 있는 위치를 나타낸다.  
여기에 나와있지 않은 함수들은 어디에서나 사용할 수 있다. 
[Context 사용 범위](https://docs.github.com/en/actions/learn-github-actions/contexts#context-availability)

ex) toJSON 함수를 이용해서 context의 내용을 로그로 출력하기 
```
name: Context testing
on: push

jobs:
  dump_contexts_to_log:
    runs-on: ubuntu-latest
    steps:
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: echo "$GITHUB_CONTEXT"
      - name: Dump job context
        env:
          JOB_CONTEXT: ${{ toJson(job) }}
        run: echo "$JOB_CONTEXT"
      - name: Dump steps context
        env:
          STEPS_CONTEXT: ${{ toJson(steps) }}
        run: echo "$STEPS_CONTEXT"
      - name: Dump runner context
        env:
          RUNNER_CONTEXT: ${{ toJson(runner) }}
        run: echo "$RUNNER_CONTEXT"
      - name: Dump strategy context
        env:
          STRATEGY_CONTEXT: ${{ toJson(strategy) }}
        run: echo "$STRATEGY_CONTEXT"
      - name: Dump matrix context
        env:
          MATRIX_CONTEXT: ${{ toJson(matrix) }}
        run: echo "$MATRIX_CONTEXT"
```

어떤 컨텍스트를 사용할 지는 [컨텍스트 목록](https://docs.github.com/ko/actions/learn-github-actions/contexts#about-contexts)을 참조해서 사용하자. 

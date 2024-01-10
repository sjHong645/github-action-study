## 개요 

변수(Variable)는 덜 민감한 설정 정보를 저장하고 재사용할 수 있도록 한다. 
변수는 workflow를 실행하는 machine에서 사용할 수 있다. 
Action 또는 workflow의 step에서 동작하는 command에서 변수를 생성, 읽기, 수정도 가능하다. 

변수는 2가지 종류가 있다. 

- 하나의 workflow에서 사용하는 환경변수(environment variable) 
- 여러 workflow에서 사용할 수 있는 설정변수(configuration variable)

## 하나의 workflow에서 사용할 환경변수 정의 

workflow 파일에서 `env` key를 사용해서 정의할 수 있다. 
정의된 변수는 어디에 정의되어 있느냐에 따라 적용되는 범위가 달라진다. 

- workflow 파일의 맨 위에서 `env`에 의해 정의 : workflow 전체 
- `jobs.<job_id>.env`를 사용한 정의 : workflow 내에 있는 job
- `jobs.<job_id>.steps[*].env` 를 사용한 정의 : job 내의 특정 step

```
name: Greeting on variable day

on:
  workflow_dispatch

env: # workflow 전체에서 적용될 환경변수 
  DAY_OF_WEEK: Monday

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env: # jobs.<job_id>.env에서 적용된 것과 동일
         # 따라서, 해당 job 내에서 적용됨 
      Greeting: Hello
    steps:
      - name: "Say Hello Mona it's Monday"
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"

        env: # jobs.<job_id>.steps[*].env에서 적용된 것과 동일
             # 따라서, 해당 step 에서 적용됨 
          First_Name: Mona 

```

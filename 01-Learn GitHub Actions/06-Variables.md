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

### 환경변수 Naming Convention 

당연히 기본적으로 제공되는 환경변수와 똑같은 이름으로 정의할 수 없고 정의한다고 하더라도 무시된다. 

새로운 변수를 설정할 때 `_경로`라는 suffix를 꼭 붙여줘야 한다. 여기서 `경로`는 해당 변수가 위치한 경로를 의미한다. 

## 여러 workflow에서 사용할 설정변수(Configuration Variable) 정의 

workflow 뿐만 아니라 organization, repository, environment 수준에서도 설정변수를 정의할 수 있다. 

만약 organization 수준의 변수가 repository 수준의 변수랑 이름이 같다면 repository 수준에서 정의한 변수가 적용된다. 각각의 org, repo, env 수준의 변수가 이름이 똑같다면 env 수준에서 정의한 변수가 적용된다.  
즉, org < repo < env 순으로 같은 이름일 경우 앞서 애기한 순서에 따라 변수가 정의된다. 

### [Repository에서 설정변수 생성](https://docs.github.com/en/actions/learn-github-actions/variables#creating-configuration-variables-for-a-repository)

비밀값 또는 변수를 어느 repo에서 만들고 싶은지에 따라 다음과 같은 권한이 필요하다. 

- 개인적인 repo : 해당 repo의 소유자
- organization repo : admin 권한
- REST API를 통해 개인적인 repo 또는 organization repo에서 변수를 만들 때 : collaborator 권한 

### [Environment에서 설정변수 생성](https://docs.github.com/en/actions/learn-github-actions/variables#creating-configuration-variables-for-an-environment)

### [Organization에서 설정변수 생성](https://docs.github.com/en/actions/learn-github-actions/variables#creating-configuration-variables-for-an-organization)

### 설정변수 제한 

각 변수의 크기는 48KB로 제한된다. Repository 당 500개의 변수를 저장할 수 있다.  
Repository의 전체 사이즈 제한은 workflow run 하나 당 256KB이다. 

## 변수값(Variable value)에 접근하기 위한 Context 사용법 

### `env` 컨텍스트를 이용한 환경변수 접근 

```
env:
  DAY_OF_WEEK: Monday

jobs:
  greeting_job:
    runs-on: ubuntu-latest
    env:
      Greeting: Hello
    steps: 
      - name: "Say Hello Mona it's Monday"
        if: ${{ env.DAY_OF_WEEK == 'Monday' }}

        # 아래 2줄은 똑같은 문장을 출력한다. 
        run: echo "$Greeting $First_Name. Today is $DAY_OF_WEEK!"
        run: echo "${{ env.Greeting }} ${{ env.First_Name }}. Today is ${{ env.DAY_OF_WEEK }}!"
        env:
          First_Name: Mona

```

### `vars` 컨텍스트를 이용한 설정변수 접근 

```
on:
  workflow_dispatch:
env:
  # Setting an environment variable with the value of a configuration variable
  # 설정변수의 값을 가지고 환경변수를 설정하는 부분
  env_var: ${{ vars.ENV_CONTEXT_VAR }}

jobs:
  display-variables:
    name: ${{ vars.JOB_NAME }}
    # You can use configuration variables with the `vars` context for dynamic jobs
    if: ${{ vars.USE_VARIABLES == 'true' }}
    runs-on: ${{ vars.RUNNER }}
    environment: ${{ vars.ENVIRONMENT_STAGE }}
    steps:
    - name: Use variables
      run: |
        echo "repository variable : $REPOSITORY_VAR"
        echo "organization variable : $ORGANIZATION_VAR"
        echo "overridden variable : $OVERRIDE_VAR"
        echo "variable from shell environment : $env_var"
      env:
        REPOSITORY_VAR: ${{ vars.REPOSITORY_VAR }}
        ORGANIZATION_VAR: ${{ vars.ORGANIZATION_VAR }}
        OVERRIDE_VAR: ${{ vars.OVERRIDE_VAR }}
        
    - name: ${{ vars.HELLO_WORLD_STEP }}
      if: ${{ vars.HELLO_WORLD_ENABLED == 'true' }}
      uses: actions/hello-world-javascript-action@main
      with:
        who-to-greet: ${{ vars.GREET_NAME }}

```

## [기본 환경변수](https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables)

해당 링크 참조 

## OS 판별 

`RUNNER_OS` 기본 환경변수와 컨텍스트의 속성 `${{ runner.os }}`을 사용해서  
서로 다른 OS에서 사용되는 하나의 workflow 파일을 만들 수 있다. 

ex) `macos-latest`에서 `windows-latest`로 OS가 성공적으로 변경되었는지 확인하는 workflow 
```
jobs:
  if-Windows-else:
    runs-on: macos-latest
    steps:
      - name: condition 1
        if: runner.os == 'Windows' # runner의 os를 check 
        run: echo "The operating system on the runner is $env:RUNNER_OS." # windows 방식의 환경변수 선언법
      - name: condition 2
        if: runner.os != 'Windows' # runner의 os를 check 
        run: echo "The operating system on the runner is not Windows, it's $RUNNER_OS." # linux 방식의 환경변수 선언법

```

## step과 job 사이에 값을 전달하기 

`GITHUB_ENV` 환경 파일은 action에서 바로 사용되거나 shell 커맨드에서 `run` 키워드를 통해 사용될 수도 있다. [자세한 내용](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-environment-variable)

어떤 job에 있는 step에서 다른 job에 있는 step으로 값을 전달하고 싶다면 `job output`으로 값을 정의해서 사용할 수 있다. `job output`을 통해 다른 job에 있는 step이 값을 참조할 수 있게 된다. [자세한 내용](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idoutputs)

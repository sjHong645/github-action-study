## 학습 목표

1. Action을 찾아내는 방법을 안다.
2. Action을 사용하는 방법을 안다.
3. Action을 원하는대로 만들어내는 방법을 안다.

## 개요
workflow에서 사용하는 action은 3개의 위치에서 정의할 수 있다. 

- workflow 파일과 동일한 repository
- 모든 공용 repository
- docker hub에 게시된 도커 컨테이너 이미지

또한 [GitHub Marketplace](https://github.com/marketplace?type=actions)를 통해 github community에서 만든 action들을 찾아볼 수 있다. 

## Workflow 편집기에서 Marketplace action 찾기 

방법은 간단하다.

1. 편집할 workflow 파일을 찾아서 편집기를 연다.
2. 그러면 아래와 같은 화면이 나타나는데 여기서 원하는 action을 선택하면 된다. 

![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/38de015e-7baf-4f13-88b4-db74b38ca6e1)

## Workflow에 Action을 추가하는 방법 

### GitHub Marketplace을 통한 작업 추가 

github marketplace에 있는 action들은 `action의 버전`과 `workflow 구문을 통해 사용하는 방법`에 대한 정보를 포함하고 있다.  
workflow를 안정적으로 유지하기 위해서는 action의 버전을 참고해야 한다. 

해당 과정들은 workflow 편집기에서 수행한 작업들이다. 

1. workflow에서 사용할 action을 찾는다. 
2. Installation 밑에 있는 copy를 클릭한다.
3. 이렇게 복사한 action을 workflow의 새로운 step에 붙여넣는다.
4. 만약, 해당 action이 입력값(input)을 원할 경우 그 값을 설정해준다.

### 동일한 repository에서 Action을 추가하는 방법

ex) repository 파일 구조 
```
|-- hello-world (repository)
|   |__ .github
|       └── workflows
|           └── my-first-workflow.yml
|       └── actions
|           |__ hello-world-action
|               └── action.yml
```

ex) workflow 파일 ⇒ 위 예시 repo에서 `/hello-world` 에 위치한 상황
```
jobs:
  my_first_job:
    runs-on: ubuntu-latest
    steps:
      # This step checks out a copy of your repository.
      - name: My first step - check out repository
        uses: actions/checkout@v4

      # This step references the directory that contains the action.
      # 해당 단계(step)에서 같은 repo에 있는 
      # .github/actions/hello-world-action 폴더에 위치한
      # action.yml을 사용하는 작업 
      - name: Use local hello-world-action
        uses: ./.github/actions/hello-world-action
```

`action.yml` 파일은 action에 대한 메타데이터를 제공하기 위해 사용된다.  
자세한 내용 : [Metadata syntax for GitHub Actions](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions)

### 다른 repository에서 Action을 추가하는 방법 

앞서 얘기한 `동일한 repository에서 Action을 추가하는 방법`과 동일한 원리를 적용하면 된다. 

ex) 추가하고자 하는 action의 정보
- owner 이름 : other-owner
- repository 이름 : other-repo

이라고 하면 다음과 같이 workflow를 작성해줘야 한다.

ex) 
```
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: other-owner/other-repo@v3 # {소유자}/{repo이름}@{ref}
```

### Docker Hub에 있는 컨테이너를 참조하는 방법 

action이 docker 컨테이너 이미지를 통해 정의되었다면 다음과 같은 구문을 이용해서 action을 참조해야 한다.

- 구문
```
docker://{이미지이름}:{태그}
```

ex) 
```
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://alpine:3.8 # alpine이라는 이미지의 3.8 태그를 참조
```

## 사용자 지정 action에 대한 release 관리 사용 

action을 만들었다면 tag, SHA, branch를 이용해서 action의 release를 관리할 수 있다. 

### tag 사용 

- 장점 : 주(major) 버전과 부(minor) 버전 간의 전환시기를 결정할 때 유용함
- 단점 : tag 값들은 일시적이라서 관리자에 의해 변경되거나 삭제될 수 있다. 

ex) action에 `v1.0.1` 태그를 지정한 예시 
```
steps:
  - uses: actions/javascript-action@v1.0.1
```

### SHA 사용

- 장점 : 더욱 신뢰성 있는 버전 작업이 필요할 때 좋다. SHA값은 불변이라서 tag와 branch 보다 훨씬 신뢰성이 높다.
- 단점 : action에 대한 업데이트를 자동으로 수신하지 못한다. 

또한, 커밋의 SHA 전체 값을 사용해야 하고 SHA를 사용할 때 해당 값이 action의 repo에서 가져온건지 repo의 fork를 통해서 가져온건 아닌지 확인해야 한다. 

ex) 
```
steps:
  - uses: actions/javascript-action@a824008085750b8e136effc585c3cd6082bd575f
```

### branch 사용 

action의 대상에 branch를 지정하면 해당 branch의 현재 버전이 실행된다.  
이 방식은 지정한 branch가 업데이트 된 경우 문제가 발생할 수 있다. 

ex) main이라는 branch를 사용한 예시 
```
steps:
  - uses: actions/javascript-action@main
```

## Action에서 입/출력값(input, output) 사용하기 

Action의 입력값(input)과 출력값(output)을 보려면 repo의 root 디렉토리에 있는 `action.yml` 또는 `action.yaml` 파일을 봐야 한다. 

ex) 예시 action.yml 파일 
```
name: "Example"
description: "Receives file and generates output"

# inputs을 통해서 필요로 하는 입력값을 지정한다.
# 여기서는 file-path라는 이름의 입력값으로 입력값을 정의했다. 
inputs: 
  file-path: # id of input
    description: "Path to test script"
    required: true
    default: "test-file.js" # file-path라는 입력값의 default 값

# outputs을 통해 출력값을 지정한다.
# 여기서는 results-file이라는 이름으로 출력값을 정의했다. 
outputs:
  results-file: # id of output
    description: "Path to results file" 
```




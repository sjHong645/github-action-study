자동으로 Github Actions Workflow를 동작시키는 방법에 대해 알아본다. 

## workflow trigger란? 

`workflow trigger`는 workflow를 동작시키도록 하는 이벤트다. 

- 이벤트 목록  
  1. workflow의 repository에서 발생한 이벤트
  2. Github 외부에서 발생하고 Github에서 repository_dispatch 이벤트를 발생시키는 이벤트
  3. 예약된 시간
  4. 수동

workflow의 trigger는 `on` 키워드로 정의된다. [workflow syntax에 대한 자세한 내용](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on)

- workflow 동작을 trigger 발생 단계
  
  1. repository에서 event가 발생. 해당 이벤트는 commit SHA와 Git ref와 연관이 있음.
  2. Github는 `.github/workflows` 디렉토리에 commit SHA와 Git ref와 연관된 workflow 파일을 찾는다.
  3. trigger되는 이벤트와 맞는 `on:`값을 가진 workflow를 trigger한다.
 
[workflow에서 workflow 트리거하는 방법](https://docs.github.com/ko/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow)


## workflow를 trigger하는 이벤트 사용 방법 

`on` key를 이용해서 어떤 이벤트에서 workflow를 trigger할 지 지정할 수 있다. 

### 1가지 event 

ex) workflow의 repository의 branch에 push event가 발생했을 때 trigger하도록 설정 
```
on : push 
```

### 여러 가지 event 

ex) workflow의 repository의 branch에 push event 또는 fork event가 발생했을 때 trigger 하도록 설정 
```
on : [push, fork]
```

### activity 타입 과 필터를 이용한 여러 가지 event 

[activity type](https://docs.github.com/ko/actions/using-workflows/triggering-a-workflow#using-event-activity-types) / [filter](https://docs.github.com/ko/actions/using-workflows/triggering-a-workflow#using-filters)
ex) 다음과 같은 상황에 trigger가 되도록 설정했다.
- label이 만들어졌을 때
- main 브랜치에 push가 실행되었을 때
- Github Pages-enabled 브랜치에 push가 실행되었을 때 
```
on:
  label:
    types:
      - created
  push:
    branches:
      - main
page_build:
```








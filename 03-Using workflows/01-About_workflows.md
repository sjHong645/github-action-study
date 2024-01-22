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

Github는 미리 설정된 starter workflow를 제공해서 나만의 CI(Continuous Integration) 재가


자동으로 Github Actions Workflow를 동작시키는 방법에 대해 알아본다. 

## workflow trigger란? 

`workflow trigger`는 workflow를 동작시키도록 하는 이벤트다. 

- 이벤트 목록  
  1. workflow의 repository에서 발생한 이벤트
  2. Github 외부에서 발생하고 Github에서 repository_dispatch 이벤트를 발생시키는 이벤트
  3. 예약된 시간
  4. 수동

workflow의 trigger는 `on` 키워드로 정의된다. [workflow syntax에 대한 자세한 내용](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on)


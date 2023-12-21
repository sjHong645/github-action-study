## 개요 

GitHub Actions는 
- 빌드, 테스트 및 배포 파이프라인을 자동화할 수 있는 CI/CD(Continuous Integration And Continuous Delivery) 플랫폼
- 이를 통해 모든 PR(Pull Request)를 빌드 및 테스트할 수 있고 merge된 PR을 production에 배포할 수 있다.
- DevOps에 그치지 않고 저장소에서 다른 이벤트가 발생할 때 workflow를 실행할 수 있도록 한다.
- Linux, Windows, macOS 가상 머신을 제공해서 workflow를 실행하거나 자체 데이터 센터 또는 클라우드 인프라에서 자체 호스팅된 runner를 호스팅할 수 있다.

## GitHub Actions의 구성 요소 

GitHub Action Workflow는 이벤트(ex. PR이 open, Issue 생성)가 발생될 때 동작하도록 할 수 있다.  
Workflow에는 순차적 또는 병렬적으로 실행할 수 있는 하나 이상의 작업(Job)이 포함되어 있다.  

각 작업은 
  - 가상 머신 runner 또는 container 내부에서 실행할 수 있다. 
  - 사용자가 정의한 스크립트를 실행하거나 어떤 동작을 실행하는 하나 이상의 단계(step)가 있다.
     - 단계(step)는 workflow를 단순화시킬 수 있는 재사용가능한 확장이다. 

![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/505aed26-1bc8-44f5-9b91-08ce29545de1)

### Workflow : 하나 이상의 작업을 실행하는 자동화 프로세스 

- YAML 파일 형식으로 정의됨
- 수동 또는 정해진 시간에 따라 workflow가 동작할 수 있음
- `.github/workflows` 폴더에 정의되어 있어야 한다. 즉, 하나의 repo가 여러 개의 workflow를 가질 수 있다.
- workflow 내에서 다른 workflow를 참조할 수 있다. [자세한 내용](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [workflow에 대한 자세한 내용](https://docs.github.com/en/actions/using-workflows)

### Event : repo에서 workflow를 실행하도록 하는 작업 

ex) PR 생성 / Issus open / commit push 

[workflow를 실행할 수 있도록 하는 모든 이벤트 목록](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

### Jobs

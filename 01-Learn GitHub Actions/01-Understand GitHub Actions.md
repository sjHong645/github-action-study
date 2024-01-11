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

### Jobs : 동일한 runner에서 실행되는 workflow에 있는 단계(step)의 집합 

- 각각의 단계(step)는 실행될 shell script이거나 동작할 action이다.
- 단계(step)는 순서대로 실행되고 서로 종속적이다. 그리고 동일한 runner에서 실행되기 때문에 한 단계에서 다른 단계로 데이터를 공유할 수 있다.
- 기본적으로 `Job`은 종속성이 없고 서로 병렬적으로 실행된다.
- 하지만, job의 종속성을 다른 job과 함께 구성할 수 있다. 만약에 어떤 job이 다른 job에 종속되면 해당 job이 완료될 때 까지 기다렸다가 실행할 수 있다.

- [job에 대한 자세한 내용](https://docs.github.com/en/actions/using-jobs)

### Action : 복잡하지만 자주 반복되는 작업을 수행하는 GitHub Action 플랫폼의 사용자 지정 응용프로그램 

- Action을 사용해서 반복되는 코드의 양을 줄일 수 있다. 
ex) GitHub으로 부터 git 저장소를 가져오기 / 빌드 환경에서 toolchain 설정하기 / 클라우드 공급자에게 인증 설정하기 

- 직접 만들거나 GitHub Marketplace에서 사용할 action을 찾을 수 있음
- [Action에 대한 자세한 내용](https://docs.github.com/en/actions/creating-actions)

### Runners : workflow가 실행될 때 그 workflow을 실행하는 서버

- 각 runner는 한 번에 하나의 job만 실행할 수 있다.
- 다른 OS가 필요하거나 특정 HW 구성을 필요로 할 때, 직접 runner를 호스팅할 수 있다.

- [Hosting your own runners](https://docs.github.com/en/actions/hosting-your-own-runners)

### [예시 workflow](https://github.com/sponbob-pat/github-action-study/blob/main/.github/workflows/learn-github-action.yml)

### 예시 workflow를 그림으로 표현 

![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/fbc1f474-6747-4bd1-b363-8c2285074d05)



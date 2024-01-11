## 개요 

Github Actions의 주요 CI 기능 중 일부를 에시로 보여주는 workflow를 소개할 것이다. 
이 workflow가 작동하면 script를 실행해서 GitHub Docs site가 깨진 링크(broken links)를 갖고 있는지 체크한다. 

- workflow의 전체적인 실행 단계(steps)와 job 안에서 step이 어떻게 동작하는지 보여주는 그림

![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/e4266a1f-3ecc-4446-93ea-c5a356c459e9)


- 예시 workflow  
이 파일의 [최신 버전](https://github.com/github/docs/blob/main/.github/workflows/check-broken-links-github-github.yml)

아래 workflow는 설명서의 모든 페이지 내용을 렌더링하고 모든 내부 링크가 올바르게 연결되어 있는지 확인한다. 

```
# workflow의 이름을 정의한다. Github Repository > Action 탭에 정의한 이름이 나타난다. 
name: 'Link Checker: All English'

# 'on' key를 통해 workflow가 동작하는 event를 정의한다.
# 여러 개의 event 들을 정의할 수 있다. 
on:
  # Github API, CLI, 브라우저 인터페이스를 통해 workflow를 직접 동작시킬 수 있도록 설정
  workflow_dispatch:

  # commit이 main 이라는 브랜치에 push될 때 마다 workflow가 자동으로 실행되도록 설정함. 
  push:
    branches:
      - main

  # PR이 만들어지거나 수정될 때 마다 workflow가 자동으로 실행되도록 설정됨
  pull_request:


# GITHUB_TOKEN에 부여되는 기본 허가(permission)를 수정한다.
# 여기서는 contents: read permission과 `pull-requests: read` permission을 수정했다. 
# `pull-requests: read` permission은 이후에 나올 trilom/file-changes-action이라는 action에 필요한 permission이다.
permissions:
  contents: read
  pull-requests: read

# The `concurrency` key ensures that only a single workflow in the same concurrency group will run at the same time. 
# `concurrency.group` generates a concurrency group name from the workflow name and pull request information. The `||` operator is used to define fallback values. 
# `concurrency.cancel-in-progress` cancels any currently running job or workflow in the same concurrency group.

# concurrency는 동일한 동시성 그룹(concurrency group)에 있는 하나의 workflow만 동시에 실행되도록 설정한다.

# 예시를 통해 아래 내용을 설명하면 
# concurrency.group : 동시성 그룹의 이름을 workflow의 이름(github.workflow)과 PR 정보(github.event.pull_request.head.label)를 가지고 만들어내도록 했다.
# concurrency.cancel-in-progress : 같은 동시성 그룹에 있는 현재 동작중인 job 또는 workflow를 취소
concurrency:
  group: '${{ github.workflow }} @ ${{ github.event.pull_request.head.label || github.head_ref || github.ref }}'
  cancel-in-progress: true

# workflow 파일에서 동작하는 모든 job을 모아놓은 부분 ⇒ 이걸 jobs 키로 그룹지었다. 
jobs:

  # check-links라는 ID로 job을 정의함. 
  check-links:

    # runs-on : job을 실행할 machine의 유형을 지정하는 key이다. 
    # 여기서는 workflow가 github/docs-internal에 저장된 경우 self-hosted 환경에서 실행되도록 하고
    # 그렇지 않은 경우 ubuntu-latest 환경에서 실행되도록 설정했다. 
    runs-on: ${{ fromJSON('["ubuntu-latest", "self-hosted"]')[github.repository == 'github/docs-internal'] }}

    # steps key는 check-links job의 일부로써 동작하는 모든 step 들을 그룹지어 놓는다.
    steps:

      # step의 이름은 Checkout
      # actions/checkout@v4라는 action을 사용한다.
      # 이 action은 나의 repository를 check해서 runner에 repository를 다운받도록한다. 그렇게 함으로써 코드에 대한 action을 실행할 수 있도록 한다.
      # workflow에서 repository의 코드를 사용하거나 repository에서 정의한 action을 사용할 거라면 checkout action은 반드시 사용해야 한다. 
      - name: Checkout
        uses: actions/checkout@v4

      # 여기서 사용한 action의 의미는 다음과 같다. 
      # Node.js SW 패키지의 v3 버전을 설치함으로써 npm 명령어를 사용할 수 있게 되었다.
      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 16.13.x
          cache: npm

      # run key를 사용해서 runner에서 `npm ci`라는 커맨드를 실행하도록 했다.
      # 참고로 npm ci는 프로젝트에 npm SW 패키지를 설치하기 위해 사용된다. 
      - name: Install
        run: npm ci

      # trilom/file-changes-action action을 사용해서 변경된 파일을 모두 모은다. 여기서는 a6ca26c14274c33b15e6499323aac178af06ad4b라는 SHA 값을 사용해 특정 버전을 선택했다.
      # 그리고 ${{ env.HOME }}/files.json 이름의 파일을 생성한다. 
      - name: Gather files changed
        uses: trilom/file-changes-action@a6ca26c14274c33b15e6499323aac178af06ad4b
        with:
          fileOutput: 'json'

      # 파일 확인을 위해 방금 생성한 파일을 출력하도록 했다.
      # 이를 통해 workflow run의 로그를 확인할 수 있게 되면서 debug에 유용하게 사용할 수 있다. 
      - name: Show files changed
        run: cat $HOME/files.json

      # script/rendered-content-link-checker.mjs 라는 스크립트를 실행하기 위해 run key를 사용했다.
      # 또한 그에 필요한 매개변수들도 전달햇다. 
      - name: Link check (warnings, changed files)
        run: |
          ./script/rendered-content-link-checker.mjs \
            --language en \
            --max 100 \
            --check-anchors \
            --check-images \
            --verbose \
            --list $HOME/files.json

      # 위 step과 같은 원리 
      - name: Link check (critical, all files)
        run: |
          ./script/rendered-content-link-checker.mjs \
            --language en \
            --exit \
            --verbose \
            --check-images \
            --level critical

```

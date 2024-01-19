이번에는 고급 GitHub Actions 특징을 살펴볼 것이다. 마찬가지로 CI를 위해 사용한 workflow다.

## 개요 

![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/23b78b17-d475-4318-a338-1c9216aa6200)

## 파일 

``` yaml
name: Node.js Tests

on:

  workflow_dispatch:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read
  pull-requests: read

# concurrency 키를 통해 동일한 concurrency 그룹에 있는 workflow가 동시에 동작한다는 걸 보장한다.
# group : 여기서는 workflow의 이름과 PR의 정보를 가지고 그룹의 이름을 만들도록 설정함. 즉, 그룹의 이름을 적는 곳
# cancel-in-progress : 동일한 concurrency 그룹에서 동작중인 job 또는 workflow를 취소할 수 있는지 여부를 설정 (true / false)
concurrency:
  group: '${{ github.workflow }} @ ${{ github.event.pull_request.head.label || github.head_ref || github.ref }}'
  cancel-in-progress: true


jobs:
  # ID가 test인 job 
  test:

    runs-on: ${{ fromJSON('["ubuntu-latest", "self-hosted"]')[github.repository == 'github/docs-internal'] }}

    # job이 동작하는 최대 시간 60분. 물론 그전에 workflow가 끝나면 종료됨
    timeout-minutes: 60

    # job에 대한 build matrix를 정의하는 부분 
    strategy:
      # fail-fast를 false로 지정함으로써 Github가 어떤 matrix가 fail 되더라도 동작하고 있던 job을 취소하는 걸 막는다. 
      fail-fast: false

      # test-group이라는 이름의 matrix를 만들었다.
      # 이 값들은 앞으로 실행할 npm test에 의해 실핼될 test group의 이름이다. 
      matrix:
        test-group:
          [
            content,
            graphql,
            meta,
            rendering,
            routing,
            unit,
            linting,
            translations,
          ]
    
    steps:
      # with 키를 사용해서 action에 부가적인 정보를 제공한다. 
      - name: Check out repo
        uses: actions/checkout@v4
        with:
          lfs: ${{ matrix.test-group == 'content' }}
          persist-credentials: 'false'

      # 현재 repository가 github/docs-internal이라면, actions/github-script action을 사용한다.
      # 해당 action은 docs-early-access라는 이름의 branch가 있는지 없는지 체크한다. 
      - name: Figure out which docs-early-access branch to checkout, if internal repo
        if: ${{ github.repository == 'github/docs-internal' }}
        id: check-early-access
        uses: actions/github-script@v6
        env:
          BRANCH_NAME: ${{ github.head_ref || github.ref_name }}
        with:
          github-token: ${{ secrets.DOCUBOT_REPO_PAT }}
          result-encoding: string
          script: |
            const { BRANCH_NAME } = process.env
            try {
              const response = await github.repos.getBranch({
                owner: 'github',
                repo: 'docs-early-access',
                BRANCH_NAME,
              })
              console.log(`Using docs-early-access branch called '${BRANCH_NAME}'.`)
              return BRANCH_NAME
            } catch (err) {
              if (err.status === 404) {
                console.log(`There is no docs-early-access branch called '${BRANCH_NAME}' so checking out 'main' instead.`)
                return 'main'
              }
              throw err
            }

      # 현재 repository가 github/docs-internal인 경우
      # 이전 단계에서 확인했던 github/docs-early-access branch를 checkout 한다.
      - name: Check out docs-early-access too, if internal repo
        if: ${{ github.repository == 'github/docs-internal' }}
        uses: actions/checkout@v4
        with:
          repository: github/docs-early-access
          token: ${{ secrets.DOCUBOT_REPO_PAT }}
          path: docs-early-access
          ref: ${{ steps.check-early-access.outputs.result }}

      # 현재 repostory가 github/docs-internal인 경우,
      # shell 명령을 실행하기 위해서 run 키워드를 사용한다.
      # shell 명령의 내용은 docs-early-access repository의 폴더를 main repository 폴더로 옮긴다는 내용이다.
      # 여기서는 docs-early-access의 하위 폴더 assets, content, data 폴더를 이동시키고 나서 docs-early-access 폴더를 삭제했다. 
      - name: Merge docs-early-access repo's folders
        if: ${{ github.repository == 'github/docs-internal' }}
        run: |
          mv docs-early-access/assets assets/images/early-access
          mv docs-early-access/content content/early-access
          mv docs-early-access/data data/early-access
          rm -r docs-early-access

# This step runs a command to check out large file storage (LFS) objects from the repository.
      # run 명령어를 사용해 shell 명령을 실행한다.
      # repository에 있는 Large File Storage를 checkout 하기 위해서 실행한다.
      - name: Checkout LFS objects
        run: git lfs checkout

      # trilom/file-changes-action 액션을 사용해서 PR에서 변경된 파일을 모아서 다음 단계(step)에서 분석될 수 있도록 한다. 
      # SHA값 a6ca26c14274c33b15e6499323aac178af06ad4b로 버전을 특정했다. 
      - name: Gather files changed
        uses: trilom/file-changes-action@a6ca26c14274c33b15e6499323aac178af06ad4b
        id: get_diff_files
        with:
          output: ' '

      # PR에서 변경된 파일의 리스트를 저장한 이전 단계(step)의 결과물(output)을 사용하는 shell 명령을 실행한다. 
      - name: Insight into changed files
        run: |

          echo "${{ steps.get_diff_files.outputs.files }}" > get_diff_files.txt

      # actions/setup-node 액션을 사용해서 특정 버전의 node SW 패키지를 runner에 설치한다.
      # 해당 node SW 패키지는 npm 명령어를 사용할 수 있도록 해준다. 
      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 16.14.x
          cache: npm

      # npm ci 명령을 사용해서 프로젝트에 npm SW 패키지를 설치하도록 했다. 
      - name: Install dependencies
        run: npm ci

      # actions/cache 액션을 사용해서 Next.js 빌드를 캐시한다. 그래서 workflow는 build의 캐시를 불러와서 매번 재빌딩하지 않는다.
      - name: Cache nextjs build
        uses: actions/cache@v3
        with:
          path: .next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('package*.json') }}

      # build 스크립트를 실행 
      - name: Run build script
        run: npm run build

# This step runs the tests using `npm test`, and the test matrix provides a different value for `${{ matrix.test-group }}` for each job in the matrix. It uses the `DIFF_FILE` environment variable to know which files have changed, and uses the `CHANGELOG_CACHE_FILE_PATH` environment variable for the changelog cache file.
      # npm test를 사용해서 테스트를 실행한다.
      # test matrix는 matrix에 있는 각각의 job의 ${{ matrix.test-group }}에 대한 서로 다른 값을 제공한다.
      # DIFF_FILE이라는 환경 변수를 사용해서 어떤 파일에 변화가 생겼는지 파악하고
      # changelog 캐시 파일에 관한 CHANGELOG_CACHE_FILE_PATH라는 환경 변수를 사용한다. 
      - name: Run tests
        env:
          DIFF_FILE: get_diff_files.txt
          CHANGELOG_CACHE_FILE_PATH: src/fixtures/fixtures/changelog-feed.json
        run: npm test -- tests/${{ matrix.test-group }}/

```

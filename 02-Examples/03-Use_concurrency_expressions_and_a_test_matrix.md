이번에는 CI에서 사용하는 고급 GitHub Actions 특징을 살펴볼 것이다. 

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

# The `concurrency` key ensures that only a single workflow in the same concurrency group will run at the same time. For more information, see "[AUTOTITLE](/actions/using-jobs/using-concurrency)."
# `concurrency.group` generates a concurrency group name from the workflow name and pull request information. The `||` operator is used to define fallback values.
# `concurrency.cancel-in-progress` cancels any currently running job or workflow in the same concurrency group.
concurrency:
  group: '${{ github.workflow }} @ ${{ github.event.pull_request.head.label || github.head_ref || github.ref }}'
  cancel-in-progress: true


jobs:

# This defines a job with the ID `test` that is stored within the `jobs` key.
  test:

    runs-on: ${{ fromJSON('["ubuntu-latest", "self-hosted"]')[github.repository == 'github/docs-internal'] }}

    # job이 동작하는 최대 시간 60분. 물론 그전에 workflow가 끝나면 종료됨
    timeout-minutes: 60

# This section defines the build matrix for your jobs.
    strategy:

# Setting `fail-fast` to `false` prevents GitHub from cancelling all in-progress jobs if any matrix job fails.
      fail-fast: false

# This creates a matrix named `test-group`, with an array of test groups. These values match the names of test groups that will be run by `npm test`.
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

# This groups together all the steps that will run as part of the `test` job. Each job in a workflow has its own `steps` section.
    steps:

# The `uses` keyword tells the job to retrieve the action named `actions/checkout`. This is an action that checks out your repository and downloads it to the runner, allowing you to run actions against your code (such as testing tools). You must use the checkout action any time your workflow will use your repository's code. Some extra options are provided to the action using the `with` key.
      - name: Check out repo
        uses: actions/checkout@v4
        with:
          lfs: ${{ matrix.test-group == 'content' }}
          persist-credentials: 'false'

# If the current repository is the `github/docs-internal` repository, this step uses the `actions/github-script` action to run a script to check if there is a branch called `docs-early-access`.
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

# If the current repository is the `github/docs-internal` repository, this step checks out the branch from the `github/docs-early-access` that was identified in the previous step.
      - name: Check out docs-early-access too, if internal repo
        if: ${{ github.repository == 'github/docs-internal' }}
        uses: actions/checkout@v4
        with:
          repository: github/docs-early-access
          token: ${{ secrets.DOCUBOT_REPO_PAT }}
          path: docs-early-access
          ref: ${{ steps.check-early-access.outputs.result }}

# If the current repository is the `github/docs-internal` repository, this step uses the `run` keyword to execute shell commands to move the `docs-early-access` repository's folders into the main repository's folders.
      - name: Merge docs-early-access repo's folders
        if: ${{ github.repository == 'github/docs-internal' }}
        run: |
          mv docs-early-access/assets assets/images/early-access
          mv docs-early-access/content content/early-access
          mv docs-early-access/data data/early-access
          rm -r docs-early-access

# This step runs a command to check out large file storage (LFS) objects from the repository.
      - name: Checkout LFS objects
        run: git lfs checkout

# This step uses the `trilom/file-changes-action` action to gather the files changed in the pull request, so they can be analyzed in the next step. This example is pinned to a specific version of the action, using the `a6ca26c14274c33b15e6499323aac178af06ad4b` SHA.
      - name: Gather files changed
        uses: trilom/file-changes-action@a6ca26c14274c33b15e6499323aac178af06ad4b
        id: get_diff_files
        with:
          output: ' '

# This step runs a shell command that uses an output from the previous step to create a file containing the list of files changed in the pull request.
      - name: Insight into changed files
        run: |

          echo "${{ steps.get_diff_files.outputs.files }}" > get_diff_files.txt

# This step uses the `actions/setup-node` action to install the specified version of the `node` software package on the runner, which gives you access to the `npm` command.
      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 16.14.x
          cache: npm

# This step runs the `npm ci` shell command to install the npm software packages for the project.
      - name: Install dependencies
        run: npm ci

# This step uses the `actions/cache` action to cache the Next.js build, so that the workflow will attempt to retrieve a cache of the build, and not rebuild it from scratch every time. For more information, see "[AUTOTITLE](/actions/using-workflows/caching-dependencies-to-speed-up-workflows)."
      - name: Cache nextjs build
        uses: actions/cache@v3
        with:
          path: .next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('package*.json') }}

# This step runs the build script.
      - name: Run build script
        run: npm run build

# This step runs the tests using `npm test`, and the test matrix provides a different value for `${{ matrix.test-group }}` for each job in the matrix. It uses the `DIFF_FILE` environment variable to know which files have changed, and uses the `CHANGELOG_CACHE_FILE_PATH` environment variable for the changelog cache file.
      - name: Run tests
        env:
          DIFF_FILE: get_diff_files.txt
          CHANGELOG_CACHE_FILE_PATH: src/fixtures/fixtures/changelog-feed.json
        run: npm test -- tests/${{ matrix.test-group }}/

```

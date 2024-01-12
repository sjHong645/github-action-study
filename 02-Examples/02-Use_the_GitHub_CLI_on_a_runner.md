workflow의 내용은 [01번 문서](https://github.com/sponbob-pat/github-action-study/blob/main/02-Examples/01-Use_scripts_to_test_your_code_on_a_runner.md)에서 예시로 든 workflow 파일의 내용과 비슷하다.

차이가 있는 부분만 주석을 통해서 설명을 덧붙이도록 하겠다. 

## 개요 
![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/cb2dc56f-082c-46eb-b3cb-39e64cea9617)

## 예시 workflow 
```
name: Check all English links

# schedule 이벤트에서 cron 구문을 사용했다.
# 이를 통해 workflow가 자동으로 실행되도록 하는 정기적인 간격을 정의한다.
# 예시에서는 UTC 기준 매일 19:40에 실행되도록 했다. 
on:
  workflow_dispatch:
  schedule:
    - cron: '40 19 * * *' # once a day at 19:40 UTC / 11:40 PST

permissions:
  contents: read
  issues: write

jobs:
  check_all_english_links:
    name: Check all links
    # repository의 이름이 docs-internal 이고 organization이 github인 경우에만 check_all_english_links job이 실행되도록 했다.
    # 그렇지 않다면 check_all_english_links job은 실행되지 않는다. 
    if: github.repository == 'github/docs-internal'

    runs-on: ubuntu-latest

    # 환경변수를 설정한다.
    # GITHUB_TOKEN라는 예약 변수도 여기서 secret를 사용해서 재정의되었다.
    # 이 변수들은 workflow에서 나중에 참조될 예정이다. 
    env:
      GITHUB_TOKEN: ${{ secrets.DOCUBOT_READORG_REPO_WORKFLOW_SCOPES }}
      FIRST_RESPONDER_PROJECT: Docs content first responder
      REPORT_AUTHOR: docubot
      REPORT_LABEL: broken link report
      REPORT_REPOSITORY: github/docs-content

    steps:
      - name: Check out repo's default branch
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 16.13.x
          cache: npm
      
      # npm ci, npm run build 커맨드가 각각 따로 실행되면서 Node.js 애플리케이션을 설치 및 빌드한다. 
      - name: Run the "npm ci" command
        run: npm ci
      - name: Run the "npm run build" command
        run: npm run build

      # repository에 있는 script/check-english-links.js 파일을 실행하고 실행했을 때 출력된 결과물을 bronken_links.md 파일에 저장한다. 
      - name: Run script
        run: |
          script/check-english-links.js > broken_links.md

      # check-english-links.js 스크립트가 broken links를 발견한다면 0이 아닌값(failure)을 반환한다.
      # 그러면 broken_links.md 파일의 첫 줄의 값을 갖는 출력값을 설정할 workflow command를 사용한다. 

      # 
      # `check-english-links.js` returns 0 if no links are broken, and 1 if any links are broken. When an Actions step's exit code is 1, the action run's job status is failure and the run ends.
      #
      # The following steps create an issue for the broken link report only if any links are broken, so `if: ${{ failure() }}` ensures the steps run despite the previous step's failure of the job.
      - if: ${{ failure() }}
        name: Get title for issue
        id: check
        run: echo "title=$(head -1 broken_links.md)" >> $GITHUB_OUTPUT
      # Uses the `peter-evans/create-issue-from-file` action to create a new GitHub issue. This example is pinned to a specific version of the action, using the `ceef9be92406ace67ab5421f66570acf213ec395` SHA.
      - if: ${{ failure() }}
        name: Create issue from file
        id: broken-link-report
        uses: peter-evans/create-issue-from-file@ceef9be92406ace67ab5421f66570acf213ec395
        with:
          token: ${{ env.GITHUB_TOKEN }}

          title: ${{ steps.check.outputs.title }}
          content-filepath: ./broken_links.md
          repository: ${{ env.REPORT_REPOSITORY }}
          labels: ${{ env.REPORT_LABEL }}
      # Uses [`gh issue list`](https://cli.github.com/manual/gh_issue_list) to locate the previously created issue from earlier runs. This is [aliased](https://cli.github.com/manual/gh_alias_set) to `gh list-reports` for simpler processing in later steps.
      - if: ${{ failure() }}
        name: Close and/or comment on old issues
        env:
          NEW_REPORT_URL: 'https://github.com/${{ env.REPORT_REPOSITORY }}/issues/${{ steps.broken-link-report.outputs.issue-number }}'
        run: |
          gh alias set list-reports "issue list \
                                       --repo ${{ env.REPORT_REPOSITORY }} \
                                       --author ${{ env.REPORT_AUTHOR }} \
                                       --label '${{ env.REPORT_LABEL }}'"


          previous_report_url=$(gh list-reports \
                                  --state all \
                                  --limit 2 \
                                  --json url \
                                  --jq '.[].url' \
                                  | grep -v ${{ env.NEW_REPORT_URL }} | head -1)

          # [`gh issue comment`](https://cli.github.com/manual/gh_issue_comment) is used to add a comment to the new issue that links to the previous one.
          gh issue comment ${{ env.NEW_REPORT_URL }} --body "⬅️ [Previous report]($previous_report_url)"

          # If an issue from a previous run is open and assigned to someone, then use [`gh issue comment`](https://cli.github.com/manual/gh_issue_comment) to add a comment with a link to the new issue without closing the old report. To get the issue URL, the `jq` expression processes the resulting JSON output.
          #
          # If an issue from a previous run is open and is not assigned to anyone, use [`gh issue comment`](https://cli.github.com/manual/gh_issue_comment) to add a comment with a link to the new issue. Then use [`gh issue close`](https://cli.github.com/manual/gh_issue_close) and [`gh issue edit`](https://cli.github.com/manual/gh_issue_edit) to close the issue and remove it from the project board.

          for issue_url in $(gh list-reports \
                                  --json assignees,url \
                                  --jq '.[] | select (.assignees != []) | .url'); do
            if [ "$issue_url" != "${{ env.NEW_REPORT_URL }}" ]; then
              gh issue comment $issue_url --body "➡️ [Newer report](${{ env.NEW_REPORT_URL }})"
            fi
          done

          for issue_url in $(gh list-reports \
                                  --search 'no:assignee' \
                                  --json url \
                                  --jq '.[].url'); do
            if [ "$issue_url" != "${{ env.NEW_REPORT_URL }}" ]; then
              gh issue comment $issue_url --body "➡️ [Newer report](${{ env.NEW_REPORT_URL }})"

              # Use [`gh issue close`](https://cli.github.com/manual/gh_issue_close) to close the old issue.
              gh issue close $issue_url

              # Use [`gh issue edit`](https://cli.github.com/manual/gh_issue_edit) to edit the old issue and remove it from a specific GitHub project board.
              gh issue edit $issue_url --remove-project "${{ env.FIRST_RESPONDER_PROJECT }}"
            fi
          done

```

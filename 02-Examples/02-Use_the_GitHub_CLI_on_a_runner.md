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

      # check-english-links.js 를 실행했을 때 link가 깨지지 않았다면 0을 반환하고 그렇지 않으면 1을 반환한다.
      # Actions의 step의 exit code가 1이라면 action의 run의 job 상태는 실패(failure)가 되면서 run이 종료된다.
      # 다음 step은 깨진 링크가 존재하는 경우에만 bronken link report의 issue를 만든다. 
      # 따라서 if: ${{ failure() }}는 이전 단계의 작업 실패에도 불구하고 단계가 실행되도록 한다. 

      - if: ${{ failure() }}
        name: Get title for issue
        id: check
        run: echo "title=$(head -1 broken_links.md)" >> $GITHUB_OUTPUT # broken_links.md 파일의 첫 줄의 값을 갖는 출력값을 설정할 workflow command

      # peter-evans/create-issue-from-file라는 action을 사용한다. 이 action은 새로운 Github Issue를 생성하는 action이다.
      # 여기서는 ceef9be92406ace67ab5421f66570acf213ec395 SHA 값 버전을 갖는 action을 사용하도록 했다. 
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


      # [`gh issue list`](https://cli.github.com/manual/gh_issue_list)
      # [aliased](https://cli.github.com/manual/gh_alias_set) to `gh list-reports` for simpler processing in later steps.
      - if: ${{ failure() }}
        name: Close and/or comment on old issues
        env:
          NEW_REPORT_URL: 'https://github.com/${{ env.REPORT_REPOSITORY }}/issues/${{ steps.broken-link-report.outputs.issue-number }}'
        run: |
          # gh 명령어를 사용해서 list-reports 라는 별칭을 설정했다.
          # 별칭의 내용은 큰따옴표로 둘러싸인 내용이다.
          # 앞선 run에서 생성한 issue들을 나타내는 내용이다.
          
          # 이 코드는 두 개의 코드가 겹쳐있다.
          # 1. gh issue list \ ~~
          # 2. gh alias set list-reports "1번 내용"

          gh alias set list-reports "issue list \
                                       --repo ${{ env.REPORT_REPOSITORY }} \
                                       --author ${{ env.REPORT_AUTHOR }} \
                                       --label '${{ env.REPORT_LABEL }}'"

          # 이는 환경변수 previous_report_url를 지정하는 부분이다.
          # 지정할 값은 $(~~)이다. 
          previous_report_url=$(gh list-reports \
                                  --state all \
                                  --limit 2 \
                                  --json url \
                                  --jq '.[].url' \
                                  | grep -v ${{ env.NEW_REPORT_URL }} | head -1)

          # [`gh issue comment`](https://cli.github.com/manual/gh_issue_comment)
          # gh issue comment는 앞서 생성했던 새 issue에 댓글을 추가하기 위해 사용한다
          gh issue comment ${{ env.NEW_REPORT_URL }} --body "⬅️ [Previous report]($previous_report_url)"

          # [`gh issue comment`](https://cli.github.com/manual/gh_issue_comment)
          # [`gh issue close`](https://cli.github.com/manual/gh_issue_close)
          # [`gh issue edit`](https://cli.github.com/manual/gh_issue_edit) 

          # 만약 이전 run의 issue가 open 되어 있고 누군가에게 할당(assigned)되어 있다면 gh issue comment를 사용해서 오래된 report를 close 하지 않고
          # 새로운 issue와의 link를 첨부한 댓글을 추가할 수 있다. issue URL을 얻어내기 위해 jq 표현식이 JSON output의 결과값을 처리한다.

          # 만약 이전 run의 issue가 open 되어 있고 누군가에게 할당(assigned)되어 있지 않다면 gh issue comment를 사용해서 새 issue와의 link를 첨부한 댓글을 추가할 수 있다.
          # 그렇게 하고 나서 gh issue close와 gh issue edit 명령어를 사용해서 issue를 close하고 project board에서 삭제한다. 
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

              # 오래된 issue를 삭제하기 위해 사용함 
              gh issue close $issue_url

              # gh issue edit 명령어를 사용해서 old issue를 수정하고 project board에서 삭제한다.
              gh issue edit $issue_url --remove-project "${{ env.FIRST_RESPONDER_PROJECT }}"
            fi
          done

```

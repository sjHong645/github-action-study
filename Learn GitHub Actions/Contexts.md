## About Contexts 

컨텍스트(context) : workflow의 실행(run), 변수, runner 환경, jobs, steps에 대한 정보에 접근하는 방법 

아래 구문을 통해 context에 접근할 수 있다.
```
${{ <context> }}
```

## 컨텍스트 목록 
| Context name | Type | Description |
|---------------|------|-------------|
| `github` | `object` | Information about the workflow run. For more information, see [`github` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#github-context). |
| `env` | `object` | Contains variables set in a workflow, job, or step. For more information, see [`env` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#env-context). |
| `vars` | `object` | Contains variables set at the repository, organization, or environment levels. For more information, see [`vars` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#vars-context). |{% endif %}
| `job` | `object` | Information about the currently running job. For more information, see [`job` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#job-context). |
| `jobs` | `object` | For reusable workflows only, contains outputs of jobs from the reusable workflow. For more information, see [`jobs` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#jobs-context). |
| `steps` | `object` | Information about the steps that have been run in the current job. For more information, see [`steps` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#steps-context). |
| `runner` | `object` | Information about the runner that is running the current job. For more information, see [`runner` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#runner-context). |
| `secrets` | `object` | Contains the names and values of secrets that are available to a workflow run. For more information, see [`secrets` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#secrets-context). |
| `strategy` | `object` | Information about the matrix execution strategy for the current job. For more information, see [`strategy` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#strategy-context). |
| `matrix` | `object` | Contains the matrix properties defined in the workflow that apply to the current job. For more information, see [`matrix` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#matrix-context). |
| `needs` | `object` | Contains the outputs of all jobs that are defined as a dependency of the current job. For more information, see [`needs` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#needs-context). |
| `inputs` | `object` | Contains the inputs of a reusable or manually or manually triggered workflow. For more information, see [`inputs` context](https://docs.github.com/ko/actions/learn-github-actions/contexts#inputs-context)

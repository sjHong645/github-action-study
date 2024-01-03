## 개요

`Expression`을 사용하여 workflow 파일에서 환경변수를 프로그래밍 방식으로 설정하고 context에 접근할 수 있다. 
식은 Literal, Operator, function의 조합을 통해서 만들어낼 수 있다. 아래에서 각각에 대해 살펴보도록 하겠다. 

- 환경변수 설정 예시

```
env:
  MY_ENV_VAR: ${{ <expression> }}
```

## Literals(리터럴) 

expression의 일부로 boolean, null, number, string 데이터 타입 사용 가능

ex) 
```
env:
  myNull: ${{ null }}
  myBoolean: ${{ false }}
  myIntegerNumber: ${{ 711 }}
  myFloatNumber: ${{ -9.2 }}
  myHexNumber: ${{ 0xff }}
  myExponentialNumber: ${{ -2.99e-2 }}
  myString: Mona the Octocat
  myStringInBraces: ${{ 'It''s open source!' }}
```

boolean, null, number는 `${{`와 `}}`로 묶어줘야 한다. 

하지만, String은 `${{`와 `}}`로 묶어주지 않아도 된다. 
만약 `${{`와 `}}`로 묶는다면 작은따옴표로 감싸야 하고 작은따옴표 자체를 literal로 사용하고 싶다면 `''`와 같이 연속 2개의 `'`를 작성하면 된다. 

## Operators(연산자) 

| Operator    | Description |
| ---         | ---         |
| `( )`       | 논리적 그룹화 |
| `[ ]`       | Index |
| `.`         | 속성(Property) 참조 해제 |
| `!`         | Not |
| `<`         | Less than |
| `<=`        | Less than or equal |
| `>`         | Greater than |
| `>=`        | Greater than or equal |
| `==`        | Equal |
| `!=`        | Not equal |
| `&&`        | And |
| <code>\|\|</code> | Or |

ex) 
```
env:
  MY_ENV_VAR: ${{ github.ref == 'refs/heads/main' && 'value_for_main_branch' || 'value_for_other_branches' }}
```

위 예제는 `MY_ENV_VAR`이라는 변수를 설정하는 식이다. 

Github의 참조(Reference)가 `refs/heads/main`에 설정되었는지 여부에 따라 값이 달라진다. 

- 설정되었다면 `value_for_main_branch`로 변수값이 설정됨
- 설정되지 않았다면 `value_for_other_branches`로 변수값이 설정된다.

즉, 위 예제를 일반화하면 다음과 같다. 
```
env:
  MY_ENV_VAR: ${{ 조건 && 참일경우 저장될 값 || 거짓일경우 저장될 값 }}
```

## Functions(함수) 

Github는 Expression에서 사용할 수 있는 내장함수를 제공한다. 

#### contains

`contains(search, item)` 

`search`값이 `item`의 값을 포함한 경우 true / 그렇지 않으면 false를 반환 

ex) 문자열에서 사용
```
contains('Hello world', 'llo') ⇒ true
```

ex) object Filter에서 사용
```
contains(github.event.issue.labels.*.name, 'bug') ⇒ event와 관련된 issue가 bug라는 라벨을 가질 경우 true / 아니면 false 
```

ex) 문자열 배열에서 사용
```
contains(fromJSON('["push", "pull_request"]'), github.event_name) ⇒ github.event_name 이라는 변수가 push 이거나 pull_request인 경우 true / 아니면 false 
```

#### startsWith

`startsWith( searchString, searchValue)` 

`searchString`값이 `searchValue`값으로 시작하는 경우 true. 대소문자를 구분하지 않고 값은 문자열로 캐스팅된다. 

ex) 
```
startsWith('Hello world', 'He') ⇒ true.
```

#### endsWith

`endsWith( searchString, searchValue)` 

`searchString`값이 `searchValue`값으로 끝나는 경우 true. 대소문자를 구분하지 않고 값은 문자열로 캐스팅된다. 

ex) 
```
endsWith('Hello world', 'ld') ⇒ true.
```

#### format

예시를 보면 곧바로 이해할 수 있다.

ex1) 
```
format('Hello {0} {1} {2}', 'Mona', 'the', 'Octocat') ⇒ 'Hello Mona the Octocat'.

# { 또는 }를 출력하고 싶은 경우 ⇒ 연속 2번 쓰면 된다. 
format('{{Hello {0} {1} {2}!}}', 'Mona', 'the', 'Octocat')  ⇒ '{Hello Mona the Octocat!}'.
```

#### join 

`join(배열, 연결문자)`

ex) github.event.issue.labels.*.name 배열이 [`bug`, `help wanted`]라고 하자. 
```
join(github.event.issue.labels.*.name, ', ') ⇒ 'bug, help wanted' 
```

위와 같이 각 배열에 있는 값을 `, `로 연결하여 하나의 문자열을 만드는 함수가 join이다. 

#### toJSON

`toJSON(value)` 

`value`를 예쁘게 모양을 잡은 JSON 형식으로 표현한 값을 출력한다. 이걸 가지고 context에 주어진 정보를 디버깅할 수 있다. 

#### fromJSON 

value에 대한 JSON object 또는 json 데이터 타입을 반환한다. 
이 함수를 통해 JSON object를 evaluated expression으로 제공하거나 
문자열, bool, null 값, 배열 및 개체 등 JSON 또는 자바스크립트로 표현될 수 있는 모든 데이터 형식을 변환할 수 있다. 

ex) JSON Object를 반환하는 예제
```
name: build
on: push
jobs:
  job1: # 여기서 JSON matrix를 설정한다. 
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: echo "matrix={\"include\":[{\"project\":\"foo\",\"config\":\"Debug\"},{\"project\":\"bar\",\"config\":\"Release\"}]}" >> $GITHUB_OUTPUT
  job2: 
    needs: job1
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJSON(needs.job1.outputs.matrix) }} # job1에서 만든 matrix를 json Object로 사용하기 위해 사용했다.
    steps:
      - run: build
```

ex) JSON 데이터 타입을 반환하는 예제 
```
name: print
on: push
env:
  continue: true
  time: 3
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - continue-on-error: ${{ fromJSON(env.continue) }} # continue라는 환경변수를 boolean값으로 변환하기 위해 사용
        timeout-minutes: ${{ fromJSON(env.time) }} # time이라는 환경변수를 integer값으로 변환하기 위해 사용
        run: echo ...
```

#### hashFiles

`hashFiles(path)`

`path`라고 지정한 패턴과 일치하는 파일들에 대한 단일 해시를 반환한다. 
`path`는 `GITHUB_WORKSPACE` 디렉토리에 상대적이며 `GITHUB_WORKSPACE` 내부의 파일만 포함할 수 있다. 

### 상태 체크 함수 

#### success

#### always 

#### cancelled

#### failure

### Object Filters 

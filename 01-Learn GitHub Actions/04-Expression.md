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

이 함수는 path에 입력한 패턴에 맞는 파일에 대해 `개별 SHA-256 해시를 계산`해서 
최종적으로 `파일 집합에 대한 SHA-256 해시`를 계산한다. [SHA-256에 대한 내용](https://en.wikipedia.org/wiki/SHA-2)

[패턴 관련 내용](https://www.npmjs.com/package/@actions/glob#patterns)

ex) 단일 패턴
`hashFiles('**/package-lock.json')` ⇒ repo에 있는 `package-lock.json` 파일을 가리킨다. 

ex) 복합 패턴
`hashFiles('**/package-lock.json', '**/Gemfile.lock')` ⇒ repo에 있는 `package-lock.json`과 `Gemfile.lock` 파일에 대한 hash값 계산 후 파일 집합에 대한 hash값을 계산한다. 

### 상태 체크 함수 

#### success 

이전 step들이 모두 성공한 경우 true를 반환

ex) 
```
steps:
  ...
  - name: The job has succeeded
    if: ${{ success() }}
```

#### always 

항상 true를 출력한다. 

그래서 job이 취소된 경우에도 실행하고 싶은 step 또는 task에서 사용하는 것이 좋다. 

```
if: ${{ always() }}
```

#### cancelled

workflow가 취소된 경우 true를 반환한다. 

```
if: ${{ cancelled() }}
```

#### failure

job의 이전 step이 `실패`한 경우 `true를 반환`한다. 
의존적인 job이 있을 때 이전 job이 실패한 경우에도 `true를 반환`한다. 

```
steps:
  ...
  - name: The job has failed
    if: ${{ failure() }}
```

- 조건문에 사용되는 failure

실패 이후에도 동작할 step에 대한 추가 조건을 포함할 수 있다. 
하지만, `success()`의 기본 상태 확인을 재정의하려면 여전히 `failure()` 을 포함시켜야 한다. 

만약 상태 확인 함수(status check function)를 포함하지 않고 if문의 조건문을 작성했다면 
`success()` 메소드가 자동으로 해당 조건문에 삽입되기 때문이다. 

```
steps:
  ...
  - name: Failing step
    id: demo
    run: exit 1
  - name: The demo step has failed
    if: ${{ failure() && steps.demo.conclusion == 'failure' }}
```

### Object Filters 

ex) `fruits`라는 이름의 object의 배열
```
[
  { "name": "apple", "quantity": 1 },
  { "name": "orange", "quantity": 2 },
  { "name": "pear", "quantity": 1 }
]
```

이때, `fruits.*.name`이라는 필터는 `["apple", "orange", "pear" ]`를 반환한다. 
말 그대로 `fruits`라는 배열의 `name`과 매핑된 값만 필터링했다. 

ex) `vegetables`라는 이름의 object 배열 
```
{
  "scallions":
  {
    "colors": ["green", "white", "red"],
    "ediblePortions": ["roots", "stalks"],
  },
  "beets":
  {
    "colors": ["purple", "red", "gold", "white", "pink"],
    "ediblePortions": ["roots", "stems", "leaves"],
  },
  "artichokes":
  {
    "colors": ["green", "purple", "red", "black"],
    "ediblePortions": ["hearts", "stems", "leaves"],
  },
}
```

이때, `vegetables.*.ediblePortion`라는 필터는 다음과 같은 값을 반환한다. 
```
[
  ["roots", "stalks"],
  ["hearts", "stems", "leaves"],
  ["roots", "stems", "leaves"],
]
```

각 배열에 있는 `ediblePortion`값과 매핑된 값만 필터링 했다. 

또한 object는 순서를 보장하지는 않는다. 

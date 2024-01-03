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


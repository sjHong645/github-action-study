## 학습 목표

1. Action을 찾아내는 방법을 안다.
2. Action을 사용하는 방법을 안다.
3. Action을 원하는대로 만들어내는 방법을 안다.

## 개요
workflow에서 사용하는 action은 3개의 위치에서 정의할 수 있다. 

- workflow 파일과 동일한 repository
- 모든 공용 repository
- docker hub에 게시된 도커 컨테이너 이미지

또한 [GitHub Marketplace](https://github.com/marketplace?type=actions)를 통해 github community에서 만든 action들을 찾아볼 수 있다. 

## Workflow 편집기에서 Marketplace action 찾기 

방법은 간단하다.

1. 편집할 workflow 파일을 찾아서 편집기를 연다.
2. 그러면 아래와 같은 화면이 나타나는데 여기서 원하는 action을 선택하면 된다. 

![image](https://github.com/sponbob-pat/github-action-study/assets/64796257/38de015e-7baf-4f13-88b4-db74b38ca6e1)

## Workflow에 Action을 추가하는 방법 

### GitHub Marketplace을 통한 작업 추가 

github marketplace에 있는 action들은 `action의 버전`과 `workflow 구문을 통해 사용하는 방법`에 대한 정보를 포함하고 있다. 
workflow를 안정적으로 유지하기 위해서는 action의 버전을 참고해야 한다. 

해당 과정들은 workflow 편집기에서 수행한 작업들이다. 

1. workflow에서 사용할 action을 찾는다. 
2. Installation 밑에 있는 copy를 클릭한다.
3. 이렇게 복사한 action을 workflow의 새로운 step에 붙여넣는다.
4. 만약, 해당 action이 입력값(input)을 원할 경우 그 값을 설정해준다.

### 동일한 repository에서 Action을 추가하는 방법




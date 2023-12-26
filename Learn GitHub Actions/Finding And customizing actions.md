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


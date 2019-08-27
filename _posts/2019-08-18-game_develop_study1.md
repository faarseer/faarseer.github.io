---

title: "unity 기초에 대해"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/TheOrderOfTime.png
categories: 
  - game
  - develop
tags:
  - game
  - devleop
last_modified_at: 2019-08-27 12:00

---

# unity 기초

## unity view
###  Hierarchy view
게임을 구성하는 요소들의 목록을 볼 수 있는 뷰
### Scene view
게임을 구성하는 각 요소들의 모습과 배치상태를 볼 수 있다.
Hierarchy view에서 요소를 더블 클릭하면, Scene view에서 그 게임 요소가 나오도록 화면이 이동.

###  Game view
Scene view를 촬영해서 하나의 화면으로 보는 게임 화면
### Inspector view
게임의 구성 요소들에 대한 자세한 정보를 표시
### Projector view
프로젝트를 구성하는 파일들을 표시
파일트리를 보여준다
### Console view
게임을 실행하면서 게임이 어떻게 실행되는지에 대한 로그를 표시

## Game Object
게임을 실행하면 나오는 게임을 구성하는 요소
- ex : player, obstacle, game env ...
새로운 게임 오브젝트 만들기 ( 큐브 )
- 메뉴-GameObject-3D Object-Cube
- Hierarchy view 에서 새로 만든 Game Object를 Cube라는 목록으로 표시
- Scene view에서 새로 만든 Game Object를 배치 상태와 모양으로 표시

조작을 쉽게 하는 법
1. Q : Scene view에 마우스를 누르고 드래그하면 화면이 이동
2. W : Game object의 위치를 조정
3. E : Game object를 회전
4. R : Game object의 크기를 조정
5. Alt : Alt키를 누르고 Scene view를 드래그 해서 시점을 조정
6. mouse wheel : mouse wheel 을 통한 줌인 / 줌아웃

### Inspector View로 Game Object 조작
#### transform
1. position 
2. rotation
3. scale

## Main Camera & Directional Light
### Main Camera
Game view 에서 보이는 화면을 지정하는 Game Object
옆으로 누운 피라미드 모양이다.
#### Camera in inspector View
이걸로 미세 조정 가능한테 나중에 알아본댄다.

### Directional LIght
말 그대로 일방향의 빛을 쏘는 Game Object

## Component
Game Object에 성질을 부여하는 요소
- ex : Main Camera Game object가 카메라로서 작용할 수 있는 것은 Inspector에 보이는 Camera Component 때문이다.
- ex :
### Main Camera
Game view 에서 보이는 화면을 지정하는 Game Object
옆으로 누운 피라미드 모양이다.
#### Camera in inspector View
이걸로 미세 조정 가능한테 나중에 알아본댄다.

### Directional LIght
말 그대로 일방향의 빛을 쏘는 Game Object

## Component
Game Object에 성질을 부여하는 요소
- ex : Main Camera 가 카메라로서 작용할 수 있는 이유는 Camera component를 갖고 있기 때문.

## Rigid Body
중력같은 요소, 물리 엔진을 적용하기 위한 요소, 를 Rigid body 라고 한다.

## Physical Material
collider 에 physical Material 를 적용.

## collider
말 그대로 collider

## c# scripts
원하는 component 를 추가하고 싶을 때 사용하자.

### c# 변수 선언
변수를 선언하고, 변수를 가져와서 값을 지정.
```c
int age;
age = 26;

int age = 26;
```

int float string
**string**을 만들 때 두 단어를 붙여준 변수명을 적을때는 두번째 단어의 첫 문자를 대문자로 해서 가독성을 높이자.

### 변수 계산
integer 에서 같은 변수에 숫자 1 더하기
```c
int age = 30;
age = age +1;

age++;
```
### script log
1. MonoBehaviour{} <- 이게 클래스에서 선언되는데 이건 어떤 타입??
2. public class $Name:MonoBehaviour{} 이것 자체가 클래스임.
3. void Start(){} <- 게임 시작될 때 실행 <- method
4. void Update(){} <- 매 프레임마다 실행 <- method
5. log 를 확인하기 위해서.. Debug.Log(); 를 사용해보자.

### class & method
1. method 일반적으로 첫문자 대문자
2. Start(), Updata() method는 유니티에서 특수하게 호출된다.

### 멤버변수와 지역변수
1. 클래스 안에 정의된 변수를 멤버변수(member variable)
2. 메소드 안에서 정의된 변수를 지역변수(local variable)

### Transform position
1. transform.position.x
2. ? 근데 메소드 안에서 변수 선언했는데 다른 메소드에서 호출이 되네..? 뭐야

### GetComponent 
1. Rigidbody $varname = Getcomponent<Rigidbody>();
2. 왜 Rigidbody 는 이렇게 선언하는가?? transform 은 그냥 바로 불렀는데..
3. Rigidbody.useGravity()
4. 그냥 얘가 연습하느라 그렇게 함.
5. 근데 Rigidbody 라는 타입으로 선언을 해준거는 왜 그런걸까 transform 은 선언해준게 없는뎅

### Game Object 가져오기


# 2019.08.25 
## csharp

### first script
```c
using UnityEngine;
using System.Collections;

public class Demo : MonoBehaviour{
void Start(){}

void Update(){}

} 

```
오늘은 이 스크립트에서 보게 되는 모든 것들에 대해서 씨샵 베이스 시각과 유니티로 적용해야 하는 시각, 이 두가지 시각에서 설명을 하겠다. 

들어가기에 앞서, 왜 기본 스크립트가 이 모냥일까? 왜 유니티가 씨샵을 쓰는가? 게임 개발에 앞서 게임 안의 모든 것들을 객체화 시켜서 만들어야 디버깅이나 수정에서 매우 간편하고 메모리 또한 효율적으로 사용할 수 있다. 씨샵은 객체화를 class나 struct로 사용하는데 운좋게도, struct 와 class의 차이? 씨샵에서는 없다.
> 그러나 strcut 는 상속이 불가능하고, class 는 상속이 가능하니까 게임개발을 위해서는 class type을 사용해야 한다.

#### 1. namespace 
이제 첫줄 부터 읽어보자.
using $namespace ? 클래스를 감싸는 대표 이름 이라고 생각하자.
여러 클래스를 만들고 하나의 네임스페이스로 지정해두면 나중에 그걸 쓰고 싶을 때 Using namespace를 치면된다.

```c
namespace $name{
pulic static class $name{}
} 
```
여기서 {}를 필드라고 하자.

자 이제 다시 위에꺼를 보면, 위의 Demo: MonoBehaviour 이라는 클래스이고, 그안에
start와 update라는 void함수가 있는 것을 알 수 있다. void함수는 return값이 없는 함수이니까 함수필드에서 변수값에 대한 메모리 값을 정해야 한다.

여기서 메모리 값을 정한다고? 포인터가 떠오르기 시작하는데... 다행히도 씨샵은 씨와 씨쁠쁠과 다르게 더 쉬운 언어? 이기 때문에 포인터를 지원하지 않고, 그냥 바로 지정한 변수에 값을 씌워주면 알아서 된단다. 

#### 2. variable
이제 변수에 대해 알아볼까나?
변수를 설명하기에 앞서 변수가 가질 수 있는 자료형을 알아야 한다. 씨샵이 지원하는 자료형은 아래와 같다.

| C# NAME | 기본값 | description|CLS? | .NET Type |
|:---:|::|::|::|
|sbyte|0|부호있는 8비트|NO|System.SByte|
|byte|0|부호없는 8비트||System.Byte|
|short|0|부호있는 16비트||System.Int|
|ushort|0|부호없는 16비트|NO|System.Uint16|
|int|0|부호있는 32비트||System.int32|
|uint|0|부호없는 32비트|NO|System.Uint32|
|long|0L|부호있는 64비트||System.int64|
|ulong|0L|부호없는 64비트|NO|System.Uint64|
|char|'\0'|16비트 유니코드 문자||System.Char|
|float|0.0F|32비트 부동소수점||System.Single|
|double|0.0D|64비트 부동소수점||System.Double|
|bool|False|참 거짓||System.Boolean|
|decimal|0.0M|부호있는 96비트||System.Decimal|
|string|Null|유니코드로 조합가능한 모든 문자||System.String|
|object|Null|.NET 통합 환경에 있는 모든 타입들의 기본 클래스||System.Object|

```cpp
int age = 26;
```
##### .NET ? / CLS?

닷넷이란? 현재의 인터넷 환경을 바꿀 차세대 MS의 제품군과 기술, 프로그램 개발 환경을 총칭하며,  다음 세대의 플랫폼과 서비스를 위한 프레임워크이며, 서비스로서의 소프트웨어 개발 환경이다.
CLS? common language specification 
닷 넷 컴파일러가 지원해야 하는 최소한의 표준 규약으로 닷넷 프레임워크가 언어가 반드시 지켜야 하는 언어 스펙을 따르는 언어라면 어떠한 언어라도 닷넷 프레임워크에서 실행가능하며 CLR(Common Language Runtime)이라는 독립적인 환경에서 실행된다.

 인터넷을 통해서 자유롭게 프로그램을 배포하고 개발하려면 하나의 언어만(예를 들어 C#)알고 있으면 되고, .NET framework에 의해서 동일한 컴파일된 코드를 만들 수 있다면 모든 프로그램을 .NET으로 통합할 수 있고 서비스 할 수 있다. (자바 진영의 경우 자바언어 하나만 알고 있으면 웹프로그래밍, 응용 프로그래밍, 모바일 프로그래밍 등 모두 개발이 가능한데 MS의 종래 Visual C, Visual Basic등은 그러질 못했다.)

  공통 언어 런타임(Common Language Runtime) : 소스코드를 컴파일 하면 어셈블리가 되는데, 이 어셈블리가 CLR 위에서 동작한다. 즉 CLR은 .Net Framework의 Runtime이다.  CLR은 다음과 같은 구성요소가 있다. - 공통 타입 시스템(Common Type System), 공용 언어  스펙(Common Language Specification), JIT 컴파일러(Just-In-Time Compiler), 가상 실행 시스템(Virtual Execution System)
 
 **Visual Studio.NET은 C#, Visual Basic.NET, C++, J#, Jscript 등 여러 언어들을 지원하며 닷넷 환경에서 어느 프로그래밍 언어로 작성하든 컴파일 된 코드(MSIL, Microsoft Intermediate Language)는 같다는 것이 아주 강력한 특징이다.** 즉 프로그래머가 가장 자신 있고 편한 언어로 개발하면 된다는 것이다. 그렇지만 대부분 C#을 많이 사용하고 있다. C#은 자바의 객체지향 특징과 C++의 강력함, VB처럼 쉬운 통합환경의 3가지 요소가 결합된 최고의 언어라고 할 수 있다

자, 다시 유니티로 돌아가볼까?

그면 이제 유니티 엔진이라는 네임스페이스가 MonoBehaviour 클래스를 갖고 있다는거를 알 수 있고 그러면 유니티 엔진에서 지원하는 GameObject,  Light, Camera 등등을 지원한다는 말을 뜻한다. 또한 Systems.Collections 네임스페이스를 받아서 Coroutine을 사용하게 해서 시간성을 부여해준다는 것을 알 수 있다. 또 다른 네임스페이스를 볼 수 있는 것들 중에서는 UnityEngine.UI 가 있는데 이거는 이제 새로운 유저 인터페이스 컴포넌트를 상속하고, UnityEngine.Events , UnityEngine.EventSystems 에는 더 많은 것들이 있단다. 
System과 System.IO, System.Collections.Generic 등등을 나중에 알아보자.

#### 3. class
클래스는 어떻게 지정하는건데?

접근제한자 class $name {}

처음은 접근제한자다. 내가 아는거는 public local private 인데 씨샵도 같냐?
일단 public 은 다른 스크립트들이 이 클래스를 전역변수로 인식하게 해줘서 protected나 private과는 다르게 인식한다는 점.

public: this means that other scripts will be aware of this script and can work with it.  Sometimes you might see the words “protected” or “private” instead, but those are more  advanced topics and can be skipped for now.

class: this is how I know my script is a “class”.  There are multiple types of things which can be defined, some of which include: struct, enum, and interface, but again, those are more advanced topics and can be skipped for now.

Demo : MonoBehaviour 가 조금 특이한데 둘 다 해서 이름인거냐?
Demo 부분이 이름이 된다. 이게 새로운 컴포넌트가 된다는 거지.
Demo: this is the name of the class and should also be the same name as the file we created.  Just like you can add components to GameObjects such as colliders or lights, you will now be able to add a component called “Demo” which will do something eventually… (but we haven’t implemented anything just yet)

monobehaviour를 상속받는다는 것이다.
MonoBehaviour: note that this word is separated by a colon.  It means that our script inherits from another class (by that name).  Inheritance is another advanced topic, but for now it is enough to know that you have access to a bunch of functionality because of it.
  
다른걸 더 하기전에 코드가 잘 돌아가는지 어느 부분에서 에러가 터지는 건지 알아야 겠지? 물론 너의 명석한 두뇌로 코드를 보고 약간의 에러 코드를 보고 나면 여기네 할 수 있지만, 더 쉬운 방법으로는 그냥 이상한 부분 앞에 printf 넣어서 보는 건데, 여기서는 
```cpp
Debug.Log("");
```
를 추가해서 어디까지 어떻게 돌아간건지 확인해 볼 수 있다는 것!
#### 4. variable.2
위에서 데이터 타입들에 대해 이야기 하면서 이미 변수들은 대부분 했지만, 유니티를 사용하면서 보게 될 몇몇 변수들을 정리해볼까.
일단 제일 자주 쓸 변수들의 데이터 타입은, 

> bool, int, string, float

유니티에서 쓰게 될 변수들의 데이터 타입은,

> GameObject, Transform, Vector3, RigidBody 
 
| C# NAME - Unity | 기본값 | description|CLS? | .NET Type |
|:---:|::|::|::|
|GameObject|Null|game object를 생성|||
|Transform|Null|transform을 생성|||
|Vector3|0.0f|3차원 벡터를 생성|||
|RigidBody|Null|RigidBody를 생성|||

변수를 선언하기 전에 데이터 타입 전에 코딩하는 다른 몇몇 단어들이 있는데,
> readonly

오브젝트가 생성될 때 할당되는 변수를 만들고 싶을 때

> const

비슷한데, must be initialized in the declaration only??

>static

클래스에 속하는 변수를 만들때 rather than instances of the class (for example many of the variables in the “Time” class are static).

이제 변수를 선언해 볼까?
```cpp
int age = 26;
```

```cpp
public class $name{
int age = 26;
void Update(){
int _age = 26;
}
}
```
클래스 레벨에서 선언하면 class-level scope를 주는 거다. 
이제 컴퓨터가 이 변수를 찾을 것이고 찾는 방법을 inspector 찾는 레벨을 scope 라고 생각하자.
이렇게 두면 클래스 레벨에서는 어디서나 사용할 수 있다. 메소드 내에서도 당연히 변수를 선언할 수 있지만, 그렇게 하면 메소드 레벨 scope가 될 것이다. 
이제 유니티 inspector를 알아 볼까?
유니티 인스펙터는 public 이나 [SeriallizeField] 를 유니티로 보여주니까 변수 선언할 때 조심하라구.

properties에 대해 알아보자. 알아보지 말자.
굳이 설명해야 된다면 하긴 하겠다.

이제 실전으로 들어가 볼까?
#### practice
```cpp
    using UnityEngine;
    using System.Collections;
    public class Motor : MonoBehaviour 
    {
      public float turnRate = 3.14f;
      void Update () {
        Transform.Rotate( new Vector3(0, turnRate * Time.deltaTime, 0) );
      }
    }
```
Updata 메서드는 유니티에서 매 프레임마다 호출 된다는 사실 알고 있는 것이고 y 축으로 turnRate만큼 회전한다는 것을 알 수 있다. 

#### 5. method

이제 메서드에 대해서 볼건데,
접근제한자 데이터타입 이름 (파라미터) {필드}
이거는 설명 안해도 될거 같고,
유니티의 MonoBehaviour 클래스에 만들어져 있는 메서드들에 plugin해보자.

1. void Start(){}
2. void Update(){}

위의 두개는 지겨울 꺼니까 더이상 설명하지 안겠다.

1. Awake(){}
2. OnEnable(){}
3. Start(){}
4. OnDestory(){} 

#### Generic
1. Generic List
배열의 불편함.. "immutable" : 객체를 추가하거나 지움으로써 사이즈를 재설정할 수 없는 것에 대한 불편함

Generic list 배열과 비슷하지만, dynamic shape sizing이 가능.
```cs
Using Systems.Collections.Generic;
public List<int> $name = new List<int>();
```
아래와 같이도 선언할 수 있지만, 코딩 좀 해보면 밑에 꺼처럼 하면 바보란걸.. 알거다.
```cs
public Systems.Collections.Generic.List<int> $name = new Systems.Collections.Generic.List<int>();
```

 2019 accelerator programming summer school

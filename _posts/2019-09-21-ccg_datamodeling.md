---

title: "blogccg : Data Modeling ~ Changing Turns 정리"
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
  - ccg
last_modified_at: 2019-10-05 12:00

---

# Data Modeling ~ Changing Turns.
[TOC]

## c# Constructor
```cs
public class A {
  int a;
  public A(){ // 이 메소드가 constructor
  };
}

var B = A(); // <- 이 때 사용됨.
```

## Aspect Container
"3. Data Modeling" 에 들어가기 전에  "2. Aspect Container" 부분을 살짝 보고 가겠다.

> Relationship between GameObject and MonoBehaviour. 

Gameobject가 자신에게 달려있는 component의 reference를 가져오는 방법이나, 어떤 component 에서 다른 component 의 reference를 가져오는 방법이 되게 쉽게 되어있다.
GameObject는 "Icontainer" 라는 인터페이스로 실행되고, Component는 "IAspect" 라는 인터페이스로 실행된다.

```cs

public interface IContainer {
  T AddAspect<t> (string key = null) where T : IAspect, new();
  T AddAspect<t> (T aspect, string key = null) where T : IAspect;
  T GetAspect<t> (string key = null) where T : IAspect;
  ICollection<IAspect> Aspects ();
  }

public interface IAspect {
  IContainer container {get; set;}
}
```

```cs
public interface IFoo : IAspect {
}

public class Foo : IFoo {
  public IContainer container {get; set;};
}

public class Demo : MonoBehaviour {
  void Start() {
    var container = new Container (); // IContainer 를 상속하는 Container class 선언
    container.AddAspect<IFoo> (new Foo()); // IAspect 를 제한자로 AddAspect , key value 는 IFOO
    
    IFoo foo = container.GetAspect<IFoo> (); //IFOO key value 로 GetAspect 
    Debug.Log ("Got a foo?" + (foo != null).ToString());
  }
}
```

## Data Modeling
들어가기에 앞서, Match, Player, Card 를 programming 하기 쉽게 정의하겠다.
- Match : a collection of two opponents / players, with a concept of turn-based gameplay and therefore an idea of which player is current
- Player : a participant in a match with its own set of stats(like mana) and collection of cards in various positions like a "hand" or "deck". May be controlled by a human or computer.
- Card : an entity that is owned and operated by a player. Has a variety of types such as spells, weapons, minions, and even hero cards. Each type of card can have its own set of stats.

### Control Modes & Zones
둘 다 enum 이다.
```cs
public enum ControlModes {
	Computer,
	Local,
	Remote
}
```

```cs
public enum Zones {
	Hero,
	Weapon,
	Deck,
	Hand,
	Battlefield,
	Secrets,
	Graveyard
}
```
### Player
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Player {
	public const int maxDeck = 30;
	public const int maxHand = 10;
	public const int maxBattlefield = 7;
	public const int maxSecrets = 5;

	public readonly int index; // 이 index가 Player를 구분하게 될거임.
	public ControlModes mode;
	public Mana mana = new Mana ();
	public int fatigue;
	// List의 size 를 먼저 지정.
	public List<Card> hero = new List<Card> (1); // size : 1
	public List<Card> weapon = new List<Card> (1); // size : 1 여기서 깨닫는 점 ; List 의 클래스는 List constructor에서 size를 조절하게 만들어 놓은거임.
	public List<Card> deck = new List<Card> (maxDeck);
	public List<Card> hand = new List<Card> (maxHand);
	public List<Card> battlefield = new List<Card> (maxBattlefield);
	public List<Card> secrets = new List<Card> (maxSecrets);
	public List<Card> graveyard = new List<Card> (maxDeck);
	// List의 값을 지정.
	public List<Card> this[Zones z] { //this 는 클래스의 현재 인스턴스, 여기서는 Player.
		get {
			switch (z) {
			case Zones.Hero:
				return hero;
			case Zones.Weapon:
				return weapon;
			case Zones.Deck:
				return deck;
			case Zones.Hand:
				return hand;
			case Zones.Battlefield:
				return battlefield;
			case Zones.Secrets:
				return secrets;
			case Zones.Graveyard:
				return graveyard;
			default:
				return null;
			}
		}
	}

	public Player (int index) { // Player constructor 
		this.index = index; // Player.index 가 input number가 되게.
	}
}
```
### Match
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Match {
	public const int PlayerCount = 2; // const : static 성질의 PlayerCount 변수. 

	public List<Player> players = new List<Player> (PlayerCount); // size : 2 짜리 Player 타입 list 형식의 players 변수.
	public int currentPlayerIndex; // currentPlayerIndex 변수.

	public Player CurrentPlayer {
		get {
			return players [currentPlayerIndex]; 
		}
	} // Player 타입의 CurrentPlayer 변수. get property로 players[currentPlayerIndex]를 받음.

	public Player OpponentPlayer {
		get {
			return players [1 - currentPlayerIndex];
		}
	} // Player 타입의 OpponentPlayer 변수. get property로 players[1- currentPlayerIndex]를 받음. 
	//if currnetPlayerIndex == 0, 1 - currentPlayerIndex = 1, if currentPlayerIndex == 1, 1 - currentPlayerIndex = 0.

	public Match () {
		for (int i = 0; i < PlayerCount; ++i) {
			players.Add (new Player (i));
		}
	} // Match Constructor, players 변수에 Player에서 Constructor에 지정한 index i의 Player 타입 넣음.
}
```
### Mana
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Mana {
	public const int MaxSlots = 10;

	public int spent;
	public int permanent;
	public int overloaded;
	public int pendingOverloaded;
	public int temporary;

	public int Unlocked {
		get {
			return Mathf.Min (permanent + temporary, MaxSlots);
		}
	}

	public int Available {
		get {
			return Mathf.Min (permanent + temporary - spent, MaxSlots) - overloaded;
		}
	}
}
```
### Card
base type to sub types : polymorphism .. inheritance of classes.
sub types implement 를 interface 로 만들자.
```cs
public interface IArmored{
  int armor {  get; set;}
}

public interface ICombatant{
  int attack {  get; set;}
  int remainingAttacks {  get; set;}
  int allowedAtacks {  get; set;}
}

public interface IDestructable {
  int hitPoints {  get; set;}
  int maxHitPoints {  get; set;}
}
```

## Action Systems
event-driven programming with `sorting event responders by custom criteria` and `invoking the handlers over time for the animation sequence.`

들어가기에 앞서, c# 의 generic 과 delegete 설명을 간단히 하고 가겠다.
### c# generic & delegate
generic 은 형식 매개 변수를 지정하는 것이라 보면 되겠다. class 나 method 를 만들 때, 형식 매개 변수를 지정해놓음으로써 런타임에러 같은 위험성을 배제하고 캐스팅?에서의 효율성을 가져올 수 있다.

#### generic class, generic method
알다시피 class 이건 method 이건 그 뒤에 <> 붙이면 되는거임. 
```cs
public class A<T>{
  public void hi(T input){
  }
}

public class B{
  static void Main(){
    A<int> bye1 = new A<int>();
    bye1.hi(1);
    A<string> bye2 = new A<string>();
    bye2.hi("hello");
  }
}
```
```cs
public class A{
  public void hi<T>(T input){
    }
}
```
#### generic 제약 : where
이제까지 했던 것에서 
```cs
<T> where T : 원하는 조건
```
를 붙여주면 된다.
여기서 
```cs
new()
```
가 조건으로 많이 나왔는데, 제네릭 클래스 선언의 형식 인수에 공용 매개 변수가 없는 생성자가 있어야 함을 뜻한다.
예로,
```cs
class ItemFactory<T> where T : new()
{
    public T GetNewItem()
    {
        return new T();
    }
}
```
위와 같이 클래스가 새로운 형식의 인스턴스를 만들때 주로 사용한다.

##### 객체와 인스턴스
객체와 인스턴스를 헷갈리지 말자.
```cs
public class human{
    man clon; // 클론은 man 의 객체. 클론은 아직 메모리를 차지하지 않음. 
    junseok = man(); // 준석은 man 의 인스턴스. 준석은 메모리를 차지함.
}
```
위의 예에서 알 수 있듯이 메모리를 차지하면 인스턴스다. 선언만 되면 객체이다.
결국 실체화가 되면 인스턴스인 것이다.

#### delegate
delegate 는 너무나 잘 설명되어있는 글이 있어서 일단 링크를 붙이고 시간될 때 정리하겠다. 너도 잘 모르는게 있으면 여기서 한번 찾아봐라. 설명이 정말 잘되있다.
[delegate 설명글 링크](https://mrw0119.tistory.com/19)

### Notifications
별 거 아니고, 모든 game action 이 'will' occur 이나 'did' occur 인 경우를 post 하는 Notification 을 만들어서 모든 game action 에 대해 알 수 있도록 하는 거다.
모든 game action 들에 대한 일대일 대응 Notification 값을 지정한다.
every `actions` will be implemented by `Notifications`.
anything can `post` any Notifications, and anything can `observe` any Notification, and can also `specify to only listen` to Notifications posted by a particular object, or by any objects.

```cs
// Required using statement - notifications are in their own namespace now
using TheLiquidFire.Notifications;

// Subscribe to a notification - such as on a MonoBehaviour OnEnable method
// Can optionally pass a third parameter to only listen to notifications posted by the specified object
this.AddObserver(OnDemoNotification, "Hello World");

// Unsubscribe from a notification - such as on a MonoBehaviour OnDisable method
this.RemoveObserver(OnDemoNotification, "Hello World");

// Post a notification - could occur anywhere and by anything
this.PostNotification("Hello World");

// Sample notification handler implementation
void OnDemoNotification (object sender, object args) {
	// Handle notification here
}
```
action 의 sub class 의 인스턴스만 존재. 다른 sub class의 notification의 관리를 해야하는데, 다른 sub class 의 인스턴스는 존재하지 않는 상황이므로, 생각할 수 있는 방법에는, `첫번째 generic method가 있는데, generic method 는 instance 를 parameter 로 사용할 수 없고`, 그렇기에 실제 Global 과 같은 역할의 Global class 를 만들어서 사용하기로 했다. 

```cs
public static class Global {
	public static int GenerateID<T> () {
		return GenerateID (typeof(T));
	} // GenerateID 의 generic method.. 왜 이렇게 했는가? 한번에 안하고?

	public static int GenerateID (System.Type type) {
		return Animator.StringToHash (type.Name);
	}

	public static string PrepareNotification<T> () {
		return PrepareNotification (typeof(T));
	} // generic method

	public static string PrepareNotification (System.Type type) {
		return string.Format ("{0}.PrepareNotification", type.Name);
	}

	public static string PerformNotification<T> () {
		return PerformNotification (typeof(T));
	} // generic method

	public static string PerformNotification (System.Type type) {
		return string.Format ("{0}.PerformNotification", type.Name);
	}
}
```

### Phase
All actions are performed as a sequence of phases...
there must be needed `over posting Notifications.`

```cs
public class Phase {
  #region Fields
  public readonly GameAction owner;
  public readonly System.Action<IContainer> handler;
  public Func<IContainer, GameAction, IEnumerator> viewer;
  #endregion
  #region Constructor
  public Phase (GameAction owner, System.Action<IContainer> handler) {
    this.owner = owner;
    this.handler = handler;
  }
  #endregion
  #region Public
  public IEnumerator Flow (IContainer game) {
    bool hitKeyFrame = false; // 기본값이 false, true가 되면 그 frame에 수행 할 logic이 handler 에 있다는 것.
    if (viewer != null) {
      var sequence = viewer (game, owner);
      while (sequence.MoveNext ()) { // sequence.MoveNext() 또한 false가 기본값.
        var isKeyFrame = (sequence.Current is bool) ? (bool)sequence.Current : false;
        if (isKeyFrame) { // isKeyFrame 이 true..
          hitKeyFrame = true;
          handler (game); // game object 의 method 가 실행
        }
        yield return null;
      }
    }
    if (!hitKeyFrame) { //hitKeyFrame 이 위의 viewer if 문에 의해 true 로 바뀌지 않은 경우 무조건 handler(game) 한번 실행.
      handler (game); // game object 의 method 가 실행
    }
  }
  #endregion
}
```
### Game Action
Game Action 에 대한 전반적인 context만 정의하고 subclasses에서 특정 내용들에 대해 기술하는 느낌. 
```cs  
public class GameAction {
  #region Fields & Properties
  public readonly int id;
  public Player player { get; set; }
  public int priority { get; set; } // 'orderOfPlay'의 sorting을 덮어씌울 수 있는 방법.
  public int orderOfPlay { get; set; } // card 의 실행 순서 sorting
  public bool isCanceled { get; protected set; } // protected set 으로 함으로써 다른 곳에서 정의할 필요 없는 것에 대해 추후의 programming 문제 발생안되게.
  public Phase prepare { get; protected set; }
  public Phase perform { get; protected set; }
  #endregion
  #region Constructor
  public GameAction() {
    id = Global.GenerateID (this.GetType ());
    prepare = new Phase (this, OnPrepareKeyFrame);
    perform = new Phase (this, OnPerformKeyFrame);
  }
  #endregion
  #region Public
  public virtual void Cancel () {
    isCanceled = true;
  }
  #endregion
  #region Protected
  protected virtual void OnPrepareKeyFrame (IContainer game) {
    var notificationName = Global.PrepareNotification (this.GetType ());
    game.PostNotification (notificationName, this);
  }
  protected virtual void OnPerformKeyFrame (IContainer game) {
    var notificationName = Global.PerformNotification (this.GetType ());
    game.PostNotification (notificationName, this);
  }
  #endregion
}
```

### Action System
마지막으로 이 Action System이 모든 모델의 기능을 제공해주는 class로 사용될 것이다. 다른 곳에서는 system 이나 controller 로 등장하는 것인데, 이제까지 것들의 기능을 한번에 묶어 놓은 pipe line 이라고 생각하면 되겠다.

```cs
public class ActionSystem : Aspect {
	// TODO: add additional code here
}
```

```cs
public const string beginSequenceNotification = "ActionSystem.beginSequenceNotification";
public const string endSequenceNotification = "ActionSystem.endSequenceNotification";
public const string deathReaperNotification = "ActionSystem.deathReaperNotification";
public const string completeNotification = "ActionSystem.completeNotification";
```

```cs
GameAction rootAction;
IEnumerator rootSequence;
List<GameAction> openReactions;
public bool IsActive { get { return rootSequence != null; }}
```

```cs
public void Perform (GameAction action) {
	if (IsActive) return;
	rootAction = action;
	rootSequence = Sequence (action);
}
```

```cs
public void Update () {
	if (rootSequence == null)
		return;

	if (rootSequence.MoveNext () == false) {
		rootAction = null;
		rootSequence = null;
		openReactions = null;
		this.PostNotification (completeNotification);
	}
}
```

```cs
public void AddReaction (GameAction action) {
	if (openReactions != null)
		openReactions.Add (action);
}
```

```cs
IEnumerator Sequence (GameAction action) {
	this.PostNotification (beginSequenceNotification, action);

	var phase = MainPhase (action.prepare);
	while (phase.MoveNext ()) { yield return null; }

	phase = MainPhase (action.perform);
	while (phase.MoveNext ()) { yield return null; }

	if (rootAction == action) {
		phase = EventPhase (deathReaperNotification, action, true);
		while (phase.MoveNext ()) { yield return null; }
	}

	this.PostNotification (endSequenceNotification, action);
}
```

```cs
IEnumerator MainPhase (Phase phase) {
	if (phase.owner.isCanceled)
		yield break;

	var reactions = openReactions = new List<GameAction> ();
	var flow = phase.Flow (container);
	while (flow.MoveNext ()) { yield return null; }

	flow = ReactPhase (reactions);
	while (flow.MoveNext ()) { yield return null; }
}
```

```cs
IEnumerator ReactPhase (List<GameAction> reactions) {
	reactions.Sort (SortActions);
	foreach (GameAction reaction in reactions) {
		IEnumerator subFlow = Sequence (reaction);
		while (subFlow.MoveNext ()) {
			yield return null;
		}
	}
}
```

```cs
IEnumerator EventPhase (string notification, GameAction action, bool repeats = false) {
	List<GameAction> reactions;
	do {
		reactions = openReactions = new List<GameAction> ();
		this.PostNotification(notification, action);

		var phase = ReactPhase (reactions);
		while (phase.MoveNext ()) { yield return null; }
	} while (repeats == true && reactions.Count > 0);
}
```
```cs
int SortActions (GameAction x, GameAction y) {
	if (x.priority != y.priority) {
		return y.priority.CompareTo(x.priority);
	} else {
		return x.orderOfPlay.CompareTo(y.orderOfPlay);
	}
}
```

## Changing Turns
change turns 을 change the relevant value of a match 로 만들어서 match 의 값만 0 1 0 1 0 1 순으로 가게끔 만들자.
설명에 앞서, c#의 foreach 와 as 연산자는 알고 가야겠다.
### c# foreach, as, is 연산자
```cs
list<int> list1 = new list();
public void method1(){
    foreach(i in list1){} // list1 안에 있는 i 를 다 돈다.
}
//************************************************//

public class B{}
var A;
var A = 7 as B; // 특정 class 로 casting.
```

### Data System

```cs
public class DataSystem : Aspect {
  public Match match = new Match();
}
public static class DataSystemExtensions {
  public static Match GetMatch (this IContainer game) {
    var dataSystem = game.GetAspect<DataSystem> ();
    return dataSystem.match;
  }
}
```
### Change Turn Action

```cs
public class ChangeTurnAction : GameAction {
  public int targetPlayerIndex;
  public ChangeTurnAction (int targetPlayerIndex) {
    this.targetPlayerIndex = targetPlayerIndex;
  }
}
```

### Observers

```cs
public interface IAwake {
  void Awake();
}
public static class AwakeExtensions {
  public static void Awake (this IContainer container) {
    foreach (IAspect aspect in container.Aspects()) {
      var item = aspect as IAwake;
      if (item != null)
        item.Awake ();
    }
  }
}
```

### Match System

```cs
public class MatchSystem : Aspect, IObserve {
  public void ChangeTurn () {
    var match = container.GetMatch ();
    var nextIndex = (1 - match.currentPlayerIndex);
    ChangeTurn (nextIndex);
  }
  public void ChangeTurn (int index) {
    var action = new ChangeTurnAction (index);
    container.Perform (action);
  }
  public void Awake () {
    this.AddObserver (OnPerformChangeTurn, Global.PerformNotification<ChangeTurnAction> (), container);
  }
  public void Destroy () {
    this.RemoveObserver (OnPerformChangeTurn, Global.PerformNotification<ChangeTurnAction> (), container);
  }
  void OnPerformChangeTurn (object sender, object args) {
    var action = args as ChangeTurnAction;
    var match = container.GetMatch ();
    match.currentPlayerIndex = action.targetPlayerIndex;
  }
}
```


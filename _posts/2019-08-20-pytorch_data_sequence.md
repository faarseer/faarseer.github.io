---

title: "pytorch의 data와 sequence padding을 정리해보자."
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

## pytorch의 data와 sequence padding을 정리해보자.

1.  map-style dataset , iterable-style dataset 
2.  customizing data loading order
3.  automating batch
4.  single- and multi- process data-loading
5.  automatic memory pinning

###torch.utils.data의 subclass
#### torch.utils.data.Dataset / torch.utils.data.DataLoader
얘네들이 좋아하는 python에서의 장점은 class로 가질 수 있는 객체 지향 방식 인 것 같다. 정말 모든 것을 class로 바꾸어 놔서 오히려 약간 귀찮을 정도이다. 내가 아직 파이썬을 잘 몰라서 그런 것일 수도 있지만, 지금 이러고 있는 것이 일단 하루종일 공부하다가 쉬면서 하고 있는거라 이런 푸념 정도는 이해해주길 바란다.
#####Datatype
1. map-style dataset
2. iterable-style dataset
pytorch에서 지원하는 dataset class는 torch.utils.data.Dataset에서 class로 사용할 수 있다.
그러면 torch.utils.data.Dataset 를 super class로 받아야 하나? 막상 source code에 들어가서 보면 Dataset은 그냥 클래스에 ```__init__``` 이랑 ```__getitem__```을 사용하라는 것 빼고는 어떤 것도 있지 않다. 여기서 첫번째로 이 Dataset이 얘네가 말하는 1. map-style dataset이다. python의 예전 버전부터 이어져 오던 ```__getitem__```을 사용해서 index를 지정해주는 인스턴스를 생성하면 된다.
두번째로는 python에서 베이스 클래스에 있는 dict나 set에 있는 ```__iter__```을 사용한, index가 없지만 iterable한 객체를 본뜬 dataset 이다. 이것은 이제, 기존의 dataset과 차별성을 두겠다며 torch.utils.data.IterableDataset에 구현해 놨다. 제대로 보진 않았지만 ```__iter__```을 사용할 수 있다는 것 빼고는 다른게 없는 것 같다. 그런데, ```__getitem__```이나 ```__iter__```또한 크게 다를 바 없기 때문에 둘은 다르게 거의 없다. 약간 ```__iter__```가 좀 더 상위 호환된 느낌?정도.. 굳이 iterable-style dataset을 만든 이유라면 python에서도 얻을 수 있던 dict class가 iterable함으로써 얻는 dataset으로써의 장점 정도?

#####Data Loading Order and Sampler
data loading 은 갑자기 왜 하냐고? pytorch가 장점으로 꼽는 것이 big data를 한번에 메모리에 저장하고 꺼내쓰지 않는 코드를 구현한 것인데, 위에서 dataset을 class로 객체로 만들고 dataloader로 원하는 batchsize만큼 꺼내서 쓰는 것이다. 이게 어떻게 메모리를 아낄 수 있는지는 아직도 모르겠지만.. 찾아봐야 겠다.
하여튼, dataloader에서 dataset에 있는 data를 불러오는 작업을 하는데, 여기서 두 데이터 타입에 따라 차이가 있단다.
iterable-style dataset이라면 data loading order은 자신이 직접 customizing 할 수 있다. 
map-style dataset은 torch.utils.data.Sampler 가 base class인데, 얘가 ```__iter__```을 제공해서 map-style dataset이 key value를 labeled 하는것이라 생각된다. Sampler 밑에 여러가지 sampling 방식이 있다.
#####  Loading Batched and Non-Batched Data
dataloader은 sampeling을 통해 각각의 가져온 데이터마다 collation을 적용해 배치로 내보낼 수 있다.
1. automatic batching (Default)
`batch_size`에 따라 automatic batching인지 아닌지 갈리는데 `batch_size`가 `None`이 아니면,  자동적으로 `batch_size`에 주어진 값 만큼 dataloader가 batched samples을 뱉어낸다. 아까도 말했듯이 map-style dataset에 대해서는 batch_sampler를 지정해줘서 iterable-style dataset처럼 batch를 customizing 할 수 있다. 

sampler에서 나온 indices를 통해 가져온 data sample은 다음으로 `collate_fn`을 적용해 마지막으로 배치로 나온다.
`collate_fn`을 통해서 collation할 함수를 정하는데..
map-style dataet은, index가 정의 되어있기 때문에..

``` python
for indices in batch_sampler:
	yield collate_fn([dataset[i] for i in indices])
```

iterable-style dataset은,  iterable 하기 때문에 iterator 메서드를 호출해서, next() 메서드를 사용해주면 된다.

``` python
dataset_iter = iter(dataset)
for indices in batch_sampler:
	yield collate_fn([next(dataset[i]) for _ in indices])
```
이런 느낌이라고 하면 될 것 같다.

2. Disable automatic batching
이건 그냥 `batch_size = 1` 인 경우를 생각하면 되겠다. 이 경우에는 실제로 1로 적어도 되는지는 모르겠다. (1, data)로 될 거같아서 .. `batch_size`와`batch_sampler`가 `None`인 경우라고 여기서는 말한다. 그럼 당연히 data는 개별적으로 `collate_fn`으로 들어갈 것이다. 
map-style dataset은, 
``` python
for index in sampler:
	yield collate_fn(dataset[index])
```
iterable-style dataset은,

``` python
for data in iter(dataset):
	yield collate_fn(data)
```
이런 느낌.

##### working with collate_fn
위에서 말했듯이 automatic batching 이냐 disable automatic batching 이냐 차이에 따라 sample이 호출될 때 같이 호출 되는 경우나 각각의 datasample이 호출될때마다 호출되는 차이점이 있다. 달라 이게? 다르긴 해

##### single- and multi- process data-loading 
이 부분은 일단 넘어 가겠다.

##### Memory Pinning
이것도 넘어 갈란다.

### sequence padding
> 자 이제, 이것으로 깨달은 dynamic shape input을 지원하는 rnn dataloader 를 만들 줄 알아야 겠지?
> 그러기 위해서 패딩까지 정리해보자

cnn에서도 padding 하지 않느냐? 제로 패딩으로다가 데이터 shape가 바뀌는 것을 방지하기 위해서?
rnn에서 사용하는 padding 또한 비슷한 맥락인데 생각해보면 cnn 에서는 같은 사이즈의 사진을 넣고 다음 layer 에서의 사진 사이즈를 걱정하는 거라면, rnn에서는 time sequence가 다양하다보니 처음 넣는 input의 dynamic sequence를 padding 하자는 의미이다. 그런데 얘네가 이제 padding 된 상태로 back propagation이 일어나는 거는 전체 학습에 좋지 않으므로 dynamic sequence를 좀 더 효율적이고 효과적으로 padding 해보자는 건데, 한번 정리해보자.

pack은 pad된 sequence를 묶는 것인데.. batchsize에서 time length에 따른 decreasing order를 지켜주자.이게 좀 중요한가 보다.

pack이 뭔가?
pytorch에서는 pack sequence -> rnn module -> unpack sequence 순을 따른단다.

여기서 batch_first를 안켜면, rnn 관련 다른데서도, 나는 일단 batch가 0dim 에 있는걸 좋아하기때문에,
dimension이 ( T * B * V ) ( T는 time(sequence), B는 batch,  V는 vector) 인 걸 기억하자.

#### torch.nn.utils.rnn.PackedSequence

#### torch.nn.utils.rnn.pad_sequence
sequence를 pad 하는거

#### torck.nn.utils.rnn.pack_sequence
sequence를 pack 하는거

pack이 어떤건지 알아볼까나

#### torch.nn.utils.rnn.pack_padded_sequence
variable length에서 pad 한 sequence를 pack 하는거

#### torch.nn.utils.rnn.pad_padded_sequence
variable length를 가진 packed sequence를 pad 하는거


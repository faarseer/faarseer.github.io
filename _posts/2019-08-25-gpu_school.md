---

title: "가속기 프로그래밍 정리"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/TheOrderOfTime.png
categories: 
  - GPU
  - programming
  - OpenCL
tags:
  - OpenCL
  - programming
  - GPU
last_modified_at: 2019-08-27 12:00

---

# 2019 accelerator programming summer school

## 08.19 scripts
### 병렬 처리의 기초
####  멀티코어
##### CPU의 전력소모와 전력장벽
CPU의 전력소모는 core clock frequency 에 비례한다.
CPU의 열 발산은 전력 소모에 비례한다. 당연하겠지 전기 쓰는 만큼 열 나는거 아니냐..
1. 전력 장벽
- CPU의 클락 주파수 증가에 따른 전력 소모와 열 발산의 증가
-  Mobile device 의 경우 battery life가 문제
-  성능을 위해 무한정 클락 프리퀀시를 증가 시킬수는 없음. = 전력 장벽
-  전력 장벽을 넘기 위해서 멀티 코어가 해결책이다. 

2. Instruction Level Parallelism
- 프로세서의 성능을 높이는 또 다른 방법으로 Instruction을 parallel하게 실행하는 것이다.
- 하지만 ILP(Instruction Level Parallelism)에도 장벽이 있으니...
	1. 응용 프로그램에 든 ILP는 제한적이다.
	2. 동시에 실행될 수 있는 Instruction은 많지 않다. (평균 2.8개)

3. 멀티코어
- 두 개 이상의 독립적인 프로세서를 담고 있는 칩이면 그냥 멀티코어다.
- 전력장벽과 ILP장벽의 해결책이다.
- 그냥 프로세서를 겁나 많이 달아서 해야할 일을 나눠서 하는거다.
- 멀티코어를 분류해볼까나..
	> 코어안에 종류가 같으면 homo고 종류가 다르면 hetero다. 너무나 쉽지 않은가? 그러면 그 종류는?
	CPU, DSP, GPU, FPGA, ASIC
	
	1. Homogeneous 멀티코어
	2. Heterogeneous 멀티코어  : 이 방법이 고성능 달성이 쉽고 전력 효율이 좋다는데.. 다양하니까 당연한거 아니겠는가?(자원관리용 범용 프로세서 + 계산 전용 가속기)
		- accelerator(가속기)?
			특정한 작업을 범용 CPU보다 빨리 처리하는 프로세서면 가속기죠.

##### GPU
가속기의 한 종류인 GPU를 한번 파볼까? 여기 온 이유가 가속기 프로그래밍인데 그중에서도 GPU를 사용하니까.
수천개의 간단한 코어(scalar processor)가 하나의 GPU에 집적되어 있고 응용 프로그램에 내재된 massive parallelism를 이용한다.
... 그렇다면 GPU는 여러개의 간단한 코어가 똑같은 연산을 데이터셋에 대해서 병렬로 처리한다는 것인데, 먼저 Amdahl의 법칙을 알고 들어가자. single processor 보다 multi processor 일 때 성능 향상을 계산하는 식인데, 너무나 간단하다.
$p$ : 병렬화 될 수 있는 부분의 순차 실행 시간
$n$ : 프로세서의 개수
$speedup = \frac{1}{(1-p) + \frac{p}{n}}$
원래 single processor의 실행 시간에 대해서, 병렬화 할 수 없는 부분의 시간 그대로에다가 병렬화 할 수 있는 시간을 프로세서의 개수 만큼 나누면 multi processor일 때의 실행시간의 비율로 나오는 것이다.. 너무 당연하잖아..
그렇다면 Amdahl의 법칙에서 좀 더 나아가서 왜 가속기 코어를 써야 하는지 생각해보자. 가속기 코어 하나가 두 개의 범용 코어와 가격이 같다고 가정하고 한개의 가속기 코어는 한 개의 범용 코어보다 $i$ 배 빠르다고 가정해보자.
그렇다면 $p$만큼 걸리는 시간에서 가속기 코어를 사용 할 수 있는 부분을 $a$, 가속기 코어의 개수를 $N$ 이라고 생각해 보면, 
$speedup = \frac{1}{(1-p) + \frac{a}{i*N} + \frac{1-a}{n}}$으로 적어 볼 수 있다. 

1. 멀티코어 프로그래밍
이제까지 위에서 많은 코어를 병렬로 사용할 수록 실행 시간이 줄어드는 것을 봤는데, Flops(floating-point operations per second)를 측정해보면, 멀티코어 하드웨어가 갖는 영향력은 시작일 뿐이고, 뒤는 모두 멀티코어 소프트웨어 즉, 멀티코어 프로그래밍에 의존한다는 것을 알 수 있었다. 그니까 이제 멀티코어 프로그래밍을 하러 가보자.
하지만 멀티코어 프로그래밍은 너무나 어렵다. 애초에 우리가 컴퓨터가 인식할 수 있는 언어 중 low-level 언어로 다가 갈수록 인간의 범주에서 벗어나는 것 같은데.. 우리가 잡아야 할 토끼는 Delivering high performance와 Ease of programming 아니겠는가. 이 둘은 trade-off 관계에 있지만, 누군가 해결해 주겠지 .. :(
일단은 병렬 프로그래밍 모델의 종류에 대해서 적어본다.
	1. shared memory parallel programming model
		- OpenMP
		- Pthreads
	2. Message passing parallel programming model
		- MPI
	3. Accelerator programming model
		- OpenCL
		- SnuCL
		- CUDA
		- OpenMP, OpenACC
이 중 3번을 격파해보자구..

##### CPU의 원리
일단 그전에 CPU의 원리는 알고 가야지 않겠는가.
우리가 프로그램을 실행시키면, CPU가 프로그램에 든 machine instruction을 실행하고, ALU는 산술연산과 논리연산을 처리하고 제어장치(control unit)은 주기억장치에서 instruction을 차례대로 가져와서(fetch) 해석하고(decode) 실행한다.(execute)
다음에 가져올 instruction의 주소는 CPU 내 레지스터(빠른 저장소란다.)인 프로그램 카운터(program counter)에 저장된다.
- instruction cycle
	결국 하나의 cycle을 갖는건데, 이 사이클을 컴퓨터가 꺼질때 까지 반복한다.
	> Fetch -> Decode -> Execute   

- Pipelining
	이제 이 instruction cycle을 처리하는 방식으로 하드웨어 테크닉으로 생산성을 증가시키는 방법을 소개한다. 파이프라인, 즉 컨베이어 벨트같은 역할인데, 하나의 파이프라인은,

	> IF(Instruction Fetch) -> ID(Instruction Decode) -> EX(Execute) 

	로 이루어진다. 각 pipleline이 독립적이라고 가정하고, IF,ID,EX를 처리하는 유닛이 각각 존재한다고 가정하면, 우리는 instruction을 여러개 처리해야 할때, IF를 하나 처리하고 ID가 처리되는 순간 다른 IF를 받아서 처리하고 식의 한칸씩 옆으로 간 stacked pipelines를 생각해 볼 수 있다. 

	이제 이를 CPU에 적용한다면 In-order 실행, Out-of-Order 실행이 있는데, In-Order는 많은 instruction을 fetch하고 decode를 한 순서대로 피연산자가 사용 가능하면 그에 맞는 파이프라인화 된 functional Unit으로 보내어 실행을 하는 형식이다. 여기서 레지스터에는 연산의 결과값이 쓰인다. 하지만 Out-of-Order은 나온 순서대로가 아니라 피연산자가 실행이 끝나서 빈 순간 빈 자리에 새로운 instruction이 실행되는 방식으로 실행 순서가 뒤죽박죽이 되기 때문에 reorder buffer가 결과값에 저장된다. 그 후에 fetch순서대로 레지스터에 결과값이 쓰인다. 현재 사용되는 superscalar CPU는 IF, ID 유닛도 여러개고, EX 유닛도 여러개이지만, 앞서 말했 듯 EX가 시간 상 오래 걸리고 이를 Out-of-Order로 처리하는 방식이다.

#### 병렬성
##### instrucion level parallelism
위에서 Instruction Cycle을 Out-of-Order로 병렬 처리 하는 방법에 대해 소개했기 때문에 넘어가겠다. 
##### task parallelism
태스크 마다 걸리는 시간이 다를 수 있으므로, 태스크를 병렬 처리해서 코어에 분담한다면 노는 코어가 생길 것이고 이는 practical 하지 않다.
##### data parallelism
1. Loop-level parallelsim
	같은 연산을 여러 개의 다른 데이터에 동시에 적용
2. SIMD(Single Instruction, Multiple Data)
	한 개의 instruction에서 같은 연산을 여러 개의 다른 데이터에 동시에 적용
	- GPU SIMD
		GPU의 작은 scalar processor가 instruction을 copy한 후에 같은 연산을 여러 개의 다른 데이터에 동시에 적용.
3. SPMD(Single Program, Multiple Data)
	같은 코드를 여러 개의 다른 데이터에 동시에 적용

#### process
실행되고 있는 컴퓨터 프로그램이라고 생각하면 되겠다. OS가 관리하기 위해서 이름을 붙이고 주소를 부여한 것이다.
single core가 멀티 태스킹 하는 방법은 어떤 것이 있을까? 하나의 프로그램을 실행하다가 다른 프로그램을 실행하고 그 중간값(context)를 10ms 정도의 작은 시간에서 옮기면서 병렬 조무사같이 작업을 한다면 우리 인간은 너무나 둔해서 알아차릴 수가 없을 것이다.

#### thread
말그대로 실 같은 줄기 아닐까? 운영체제에 의해 CPU 코어에 스케쥴 되는 최소 단위, instruction cycle을 계속 실행하는 최소 단위라고 생각하면 되겠다. multi-thread, multi-core이면 instruction이 덜 찬 순서대로 채워 나간다. 
하지만 하나의 프로그램을 여러 thread에 옮기면 같은 데이터에서 불러오는 부분에서 문제가 생길 수 있지 않겠나? 이부분을 synchronization 하는 것이 또 중요한 문제겠다.

#### Dependence
일단 thread건 뭐건 서로 다른 연산을 병렬화 하기 전에 서순부터 지키자. 서순에 문제가 생기면 위에서 말했듯이 데이터를 불러오는 부분에서 바로 문제가 터진다. 이 부분을 Dependence로 서순을 표기하겠다.
1. Flow(True) dependence
	> store 후에 load
2. Anti dependence
	> load 후에 store
3. Output dependence
	> store 후에 store
4. Input dependence?
	> load 후에 load
	
	얘는 진짜 dependence는 없지만 먼저 load하면 cache에 저장될 확률이 크므로 컴퓨터 입장에서 dependence 라고 분류해주자. 	

위의 경우는 모두 Loop-Independent dependence이고, 한 iteration에서 뒤따르는 iteration이 같은 메모리를 엑세스 하는 경우를 Loop-carried dependence라고 한다.

#### Synchronization
자 이제 동기화를 해보자. 서순을 지켰으니, 같은 위치로 델고 오자는 뜻이다. 
1. Barrier
2. Data race
3. Atomicity and Mutual Exclusion
4. Lock and Unlock

5. Data race 를 의도적으로 사용한 case : Busy-wait

### 천둥 사용법

ssh ap43@chundoong.snu.ac.kr
passwd 110718

### 프로젝트 시작해 볼까나
Input : 88 one-hot vector (피아노 건반 : 치면 1, 안치면 0)
Network : FC - GRU 2 layer - FC (88, 2048, 2048, 2048, 88)
Output : 88 one-hot vector (다음 시퀀스 피아노 건반)

make_seq.c : cpu 로 계산하는 코드
music_opencl.c : OpenCL 호스트 코드 작성
kernel.cl : OpenCL 커널 코드 작성
Makefile : compiler

#### pointer
특정 변수(의 주소)를 가리키는 역할을 하는 변수

main에서 한번 만들어둔 변수를 다른 함수에서도 그래도 이용하거나 함수를 통해 변경하고 싶을 때 사용하자.
같은 지역 (main) 내에 있는 변수라면 변경은 간단하지만, 같은 지역이 아닌 경우( main 외 호출된 함수) 는 해당 값을 임의의 변수로 받아 반환하는 식으로 처리한다.
이럴 때 포인터를 쓰자. 

int형 포인터 선언 : int* variable\_name
포인터 변수 : *variable_name
변수의 주소 : $variable_name

```cpp
#include<stdio.h>
int main(void){
	int num1 = 3;
	int num2 = 6;
	int* num1Pointer = &num1;
	int* num2Pointer = &num2;

	printf("%d, %d\n", num1, *num1Pointer);
	printf("%d, %d\n", num2, *num2Pointer);
	}
```
결과값
> 3,3
> 6,6

#### GRU
##### 아이디어 및 궁금증
1. 일단 궁금증 1, input 으로 time=0 one-hot vector와 생성해야 하는 output one-hot vector 숫자를 받는 데, 여기서 one-to-multi 방식을 사용한건지 아니면 output을 다시 input으로 넣어서 one-to-one(pushed by time sequence)인지 확인해야 한다.

2. 궁금증 2, 여기서 사용한 GRU cell을 직접 디버깅해서 확인해 봐야한다.
이상한 모냥이 있어

3. 아이디어 행렬 연산 할때 배치로 계산. 

4. 아이디어 행렬 연산 할때 더하기 되어있는 부분 행렬 하나로 만들어서 계산.

5. 행렬 elemental-wise multiply 빨리 하는 법 생각해보기.

##### 기존 LSTM
![Alt text](./Screenshot 2019-08-19 23:21:04.png)

$f_t = \sigma(\mathrm{W}_{xf}^T \cdot x_t + \mathrm{W}_{hf}^T \cdot h_{t-1} + b_f)$
$i_t = \sigma(\mathrm{W}_{xi}^T \cdot x_t + \mathrm{W}_{hi}^T \cdot h_{t-1} + b_i)$
$o_t = \sigma(\mathrm{W}_{xo}^T \cdot x_t + \mathrm{W}_{ho}^T \cdot h_{t-1} + b_o)$
$\tilde{c}_t = \tanh(\mathrm{W}_{x\tilde{c}}^T \cdot x_t + \mathrm{W}_{h\tilde{c}}^T \cdot h_{t-1} + b_{\tilde{c}})$
$c_t = f \otimes c_{t-1} + i_t \otimes \tilde{c}_t$ 
$y_t, h_t = o_t \otimes \tanh(c_t)$

##### GRU Cell
![Alt text](./Screenshot 2019-08-19 23:35:37.png)


$r_t = \sigma(\mathrm{W}_{xr}^T \cdot x_t + \mathrm{W}_{hr}^T \cdot h_{t-1} + b_r)$
$z_t = \sigma(\mathrm{W}_{xz}^T \cdot x_t + \mathrm{W}_{hz}^T \cdot h_{t-1} + b_z)$
$\tilde{c}_t = \tanh(\mathrm{W}_{x\tilde{c}}^T \cdot x_t + \mathrm{W}_{h\tilde{c}}^T \cdot (r_t \otimes h_{t-1}) + b_{\tilde{c}})$
$h_t = z_t \otimes h_{t-1} + (1-z_t) \otimes \tilde{c}_t$
$z_t$ 가 1일 경우 input gate가 열리고 0일 경우 forge gate가 열리게 되어있다.


### 공지
아침 07:30 - 08:30
점심 12:00 - 13:00
저녁 18:00 - 19:00

오전 08:30 ~ 12:00 / 오후 14:00 ~ 17:30

화~수 : 기본 2층 다이아몬드
목~금 : 2층 다이아몬드

## 08.20 scripts
### OpenCL 소개
디바이스 별로 다른 프로그래밍 방법을 사용하기 때문에 개발하는데 무척이나 어렵다. 그러면 ```여러개의 디바이스가 같이 있는 이종 클러스터를 관리하려면 다른 디바이스마다 다른 프로그래밍을 배워야한다!!``` 
여러분이라면 가능할 수도 있다. 하지만 이런 곳에 골머리를 썩기 싫은 프로그래머들은 이런 이종 플랫폼들을 위한 표준 병렬 프로그래밍 모델, Open Computing Language를 만들었고 줄여서 OpenCL이라고 한다. open specification으로 Kronos Group에 의해 OpenCL의 표준이 공개된다. 즉, 저기 적힌대로만 적어주면 된다는 얘기다. 지금은 2.xx 버전이 나왔지만 우리는 1.2 버전으로 배운다.
> OpenCL 의 특징으로는,
1. 데이터 병렬 프로그래밍 모델을 지원한다.
2. ANSL/ISO C99 기반 언어 사용한다.(2.0버전부터는 C++ 지원이 추가되었다.)
3. 코드 이식성(source-code portability) : 다양한 디바이스에서 실행 가능하다.
4. low-level programming language로 fine-tuning이 가능하다. 

> 하지만 한계로는,
1. 성능 이식성(performance portability)가 부족하다. ( 타깃 디바이스에 따라 최적화가 필요하다.)
2. 프로그래밍이 어렵다. low-level language :(
3. 단일 OS 내의 디바이스들만 프로그래밍 가능하다. / 이종 클러스터 내에서 프로그래밍 하기위해서는 MPI 를 같이 사용해야 한다. 
___

### CUDA란?
Compute Unified Device Architecture 약어로써 NVIDIA 에서 만든 GPU 병렬 프로그래밍 모델이다. NVIDIA device에서만 지원되기 때문에 NVIDIA device를 사용해야 하지만 이 점을 해결하려고 OpenCL이 만들어졌고, OpenCL이 만들어질 때 CUDA를 베이스로 만들었으니까 코딩에서 겹치는 부분은 상당하다.

### OpenCL의 아이디어는?
1. SPMD(Single Program, Multiple Data) : 같은 커널 코드를 다른 데이터 아이템에 동시에 실행하는 병렬성을 부여하는 것. (여러 커널 인스턴스가 각각 다른 ID를 가진다.)
2. 데이터 병렬성을 활용한다.

OpenCL 플랫폼 모델 -> OpenCL 어플리케이션 -> OpenCL 프레임워크 -> 각각의 디바이스
OpenCL 프레임워크는 OpenCL 어플리케이션을 개발하고 실행하기 위한 소프트웨어이다.
OpenLC의 플랫폼은 프레임워크마다 각각 대응된다.
예) AMD OpenCL, Intel OpenCL, NVIDIA OpenCL ...SnuCL

플랫폼? = host + device : host 는 cpu, 디바이스는 accelerator나 멀티 cpu..

플랫폼과 디바이스을 추상화 해볼까?
플랫폼 : Host processor + Main memory
디바이스 : 여러 개의 CU(Computing Unit)으로 구성. 
CU : local memory + 여러 개의 PE(Processing Element)
PE : private memory + 계산기 : 
이제 이 PE에서 실제로 코드를 실행하면서 계산을 수행한다고 보면된다.(SIMD or SPMD 방식으로)
OpenCL 어플리케이션? = 호스트 프로그램 + OpenCL 프로그램
호스트 프로그램은 호스트 프로세서에ㅓ 실행
OpenCL 프로그램은 여러 커널로 이루어져 있고 디바이스에서 실행
호스트 프로그램은 C로 작성, OpenCL API 함수를 사용해 디바이스를 관리
커널은 OpenCL C 언어로 작성, 디바이스에서 실행되는 코드의 기본 단위라고 보면된다.
호스트 프로그램이 디바이스에 큐를 주고 자기 할 일(호스트 프로그램)을 병렬로 실행할 수 있다는 것.

OpenCL C
커널 작성을 위한 프로그래밍 언어
ISO C99 기반
제약
1. 표준 C99 header 지원하지 않음
2. 함수 포인터, 재귀함수 지원하지 않음
확장
1. 벡터 데이터 타입
2. 이미지 데이터 타입
3. 주소 공간 한정자(qualifier) 글로벌인지 로컬인지 프라이빗 메모리인지
4. 동기화(synchronization) 메모리 동기화를 말한다.
5. 많은 빌트인 함수? 표준 헤더에서 골라온 것들

이제 커널을 실행해 보자
호스트 프로그램에서 커널을 실행시킬 때 n차원의 인덱스 공간을 지정한다.(3차원 까지)
워크 아이템? 인덱스 공간의 각 점마다 커널 인스턴스가 하나씩 실행된다. 워크 아이템 하나가 PE라고 생각하면 됨. 워크 아이템의 그룹인 워크 그룹이 CE라고 생각하면 된다. 고로 하나의 워크 그룹은 하나의 CU에서 실행된다. 

정리하면, 커널 코드는 데이터에 적용되는 순서가 있는 명령어이고,
인덱스 공간은 워크 아이템과 데이터를 대응시키는 거다.

실행과정 scheme
1. 호스트 프로그램
```flow
op0=>operation: 플랫폼, 디바이스 정보를 얻음
op1=>operation: 커널 코드 실행에 필요한 객체 생성
op2=>operation: 컨텍스트(context) 생성
op3=>operation: 커맨드-큐(command-queue) 생성
op4=>operation: OpenCL 프로그램 컴파일
op5=>operation: 메모리 오브젝트 생성 후 커널 코드 실행

op0->op2->op3->op4->op5

```
2, 커널 프로그램
```flow
op0=>operation: 입력 데이터를 글로벌 메모리로 보냄
op1=>operation: 커널 실행(여러 커널 인스턴스가 실행)
op2=>operation: 출력 데이터를 글로벌 메모리에서 받음 

op0->op1->op2
```

컨텍스트(context)?
커널이 실행되는 환경으로 커맨드 간 동기화 및 메모리 관리를 수행

커맨드-큐(command-queue)?
디바이스에 커맨드를 보내는 것

프로그램 오브젝트와 커널 오브젝트
OpenCL 프로그램의 바이너리 코드

메모리 오브젝트
말 그대로 메모리 관련 객체
1. 버퍼 오브젝트 : 일반적인 배열
2. 이미지 오브젝트 : 이미지 처리를 위한 특수한 오브젝트

GPU와 OpenCL 플랫폼 모델을 비교해볼까

| GPU architecture | OpenCL platform |
|:--:|:--:|
|Core = Streaming Multiprocessor(SM)|Computing Unit(CU)|
|Arithmetic Logic Unit(ALU)|Processing Elemnet(PE)|
|GPU Memory|Global Memory|
|Shared Context in SM|Local Memory|
|context in SM|Private Memory|

### OpenCL practice
각각의 단계는 모두 method로 이루어져 있다. 뭔 말이냐고? 겁나 간단하다는 말이다.
#### 1.플랫폼과 디바이스 정보 얻기
```c
//플랫폼 개수와 ID 얻기
cl_int clGetPlatformIDs(cl_uint num_entries, cl_platform_id *platforms, cl_uint *num_platforms)

//플랫폼 정보 얻기
//param name에 얻고 싶은 플랫폼 정보를 지정해준다.
//1. CL_PLATFORM_NAME : 플랫폼 이름
//2. CL_PLATFORM_VENDOR : 플랫폼 벤더 이름
//3. CL_PLATFORM_VERSION : 플랫폼이 지원하는 OpenCL 버전
//4. CL_PLATFORM_PROFILE : full profile
//5. CL_PLATFORM_EXTENSIONS : 플랫폼이 지원하는 extension 리스트

cl_int clGetPlatformInfo(cl_platform_id platform, cl_platform_info param_name, size_t param_value_size, void *param_value, size_t *param_value_size_ret)

//디바이스 개수와 ID 얻기
//device_type에 얻어올 디바이스 종류를 지정해준다.
//1. CL_DEVICE_TYPE_ALL
//2. CL_DEVICE_TYPE_CPU
//3. CL_DEVICE_TYPE_GPU
//4. CL_DEVICE_TYPE_ACCELERATOR

cl_int clGetDeviceIDs(cl_platform_id platform, cl_device_type device_type, cl_uint num_entries, cl_device_id *devices, cl_uint *num_devices)

//디바이스 정보 얻어오기
//param_name에 어떤 정보를 얻을지 지정해준다.
//1. CL_DEVICE_NAME
//2. CL_DEVICE_VENDOR
//3. CL_DEVICE_MAX_COMPUTE_UNITS
//4. CL_DEVICE_MAX_WORK_GROUP_SIZE
//5. CL_DEVICE_GLOBAL_MEM_SIZE
//6. CL_DEVICE_LOCAL_MEM_SIZE

cl_int clGetDeviceInfo(cl_device_id device, cl_device_info param_name, size_t param_value_size, void *param_value, size_t *param_value_size_ret)
```
#### 2. 커널 코드 실행에 필요한 객체 생성
```c
//context 만들기
cl_context clCreateContext(const cl_context_properties *properties, cl_uint num_devices, const cl_device_id *devices, void (CL_CALLBACK *pfn_notify)(...), void *user_data, cl_int *errcode_ret)

//command-queue 만들기
// In-order : default
// Out-of-order : properties 에 CL_QUEUE_OUT_OF_ORDER_EXEC_MODE_ENABLE 넣으면 된다.
cl_command_queue clCreateCommandQueue(cl_context context, cl_device_id device, cl_command_queue_properties properties, cl_int *errcode_ret)

```
#### 3. 컨텍스트(context) 생성
```c
cl_int clGetPlatformIDs(cl_uint num_entries, cl_platform_id *platforms, cl_uint *num_platforms)
```
#### 4. 커맨드-큐(command-queue) 생성
```c
cl_int clGetPlatformIDs(cl_uint num_entries, cl_platform_id *platforms, cl_uint *num_platforms)
```
#### 5. OpenCL 프로그램 컴파일
```c
//소스 코드로부터 program object 만들기
cl_program clCreateProgramWithSource(cl_context context, cl_uint count, const char **strings, const size_t *lengths, cl_int *errcode_ret)

//program build
cl_int clBuildProgram(cl_program program, cl_uint num_devices, const cl_device_id *device_list, const char *options, void (CL_CALLBACK *pfn_notify)(...), void *user_data)

//compile error 발생 확인
cl_int clGetProgramBuildInfo(cl_program program, cl_device_id device, cl_program_build_info param_name, size_t param_value_size, void *param_value, size_t *param_value_size_ret)

//kernel object 만들기
cl_kernel clCreateKernel(cl_program program, const cahr *kernel_name, cl_int *errcode_ret)

```
#### 6. 메모리 오브젝트 생성 후 커널 코드 실행
```c
//buffer object 만들기
cl_mem clCreateBuffer(cl_context context, cl_mem_flags flags, size_t size, void *host_ptr, cl_int *errcode_ret)

//buffer 쓰기
cl_int clEnqueueWriteBuffer(cl_command_queue command_queue, cl_mem buffer, cl_bool blocking_write, size_t offset, size_t size, const void *ptr, cl_uint num_events_in_wait_list, const cl_event *event_wait_list, cl_event *event)

//buffer 읽기
cl_int clEnqueueReadBuffer(cl_command_queue command_queue, cl_mem buffer, cl_bool blocking_Read, size_t offset, size_t size, const void *ptr, cl_uint num_events_in_wait_list, const cl_event *event_wait_list, cl_event *event)
```

#### 7. 커널 인자 설정
```c
cl_int clSetKernelArg(cl_kerenl kernel, cl_uint arg_index, size_t arg_size, const void *arg_value)

```
#### 8. 커널 실행
```c
cl_int clEnqueueNDRangeKernel(cl_command_queue command_queue, cl_kernel kernel, cl_uint work_dim, const size_t *global_work_offset, const size_T *global_work_size, const size_t *local_work_size, cl_uint num_events_in_wait_list, const cl_event *event_wait_list, cl_event *event)
```



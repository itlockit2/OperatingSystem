Chapter 6. 프로세스 동기화(Process Synchronization)
=============
협력적 프로세스는 시스템 내에서 실행 중인 다른 프로세스의 실행에 영향을 주거나 영향을 받는 프로세스이다.
이경우 공유 데이터에 대한 동시 접근은 데이터의 비일관성을 낳을 수 있다.
## 6.1. 배경(Background)
프로세스가 병렬적으로 실행되었을때 이들은 서로 비동기적으로 수행하면서 데이터를 공유할 가능성이 크다.
### Example
 * counter++
    * register1 = counter
    * register1 = register1 + 1     
    * counter = register1
  * counter--
    * register2 = counter
    * register2 = register2 - 1
    * counter = register2
  * count가 5일때 두 프로세스 실행
    * S0: producer execute register1 = counter         {register1 = 5}
    * S1: producer execute register1 = register1 + 1   {register1 = 6} 
    * S2: consumer execute register2 = counter        {register2 = 5} 
    * S3: consumer execute register2 = register2 – 1  {register2 = 4} 
    * S4: producer execute counter = register1         {counter = 6 } 
    * S5: consumer execute counter = register2        {counter = 4}
### 경쟁상황(RaceCondtion)
- 이처럼 두개의 프로세스가 동시에 변수 counter를 조작하고
그 실행결과가 접근이 발생한 특정 순서에 의존하는 상황을 경쟁 상황(Race Condition) 이라고 한다.
## 6.2 임계구역 문제(The Critical-Section Problem)
각 프로세스는 임계구역(Critical Section)이라고 부르는 코드 부분을 포함하고 있고, 그 안에서는 다른 프로세스와 공유하는 변수를 변경하거나, 테이블을 갱신
하거나 파일을 쓰거나 하는 등의 작업을 수행한다.
### 임계구역 특징
임계구역의 가장큰 특징은 한 프로세스가 자신의 임계구역에서 수행하는 동안에는 다른 프로세스들은 그들의 임계구역에 들어갈 수 없다는 사실이다.

   ![Alt text](/1.jpg)
   
 <center>  <전형적인 프로세스의 일반적인 임계구역 구조> </center> 
  
###임계구역 구조
#### 진입 구역(Entry Section)
자신의 임계구역으로 진입하려면 진입 허가를 요청해야하는데 이러한 요청을 구현하는 코드부분이다.
#### 퇴출 구역(Exit Section)
임계구역에서 빠져나올때 다른 프로세스가 임계구역에 들어갈수 있게끔 처리하는 코드부분이다.
#### 나머지 구역(Remainder Section)
임계구역과 관련없는 나머지 코드부분이다.
### 임계구역 문제 해결안
1. 상호배제(Mutual Exclusion) : 프로세스 P가 자기의 임계구역에서 실행된다면, 다른 프로세스들은 그들 자신의 임계구역에서 실행될 수 없다.
2. 진행(Progress) : 임계구역으로 진입하려고 하는 프로세스들이 있다면 나머지구역(Remainder Section)을 실행하고 있는 프로세스가 아닌
진입구역(Entry Section)을 실행중인 프로세스들만 경쟁할 수 있으며 이 선택은 무한정 연기될 수 없다.
3. 한정된 대기(Bounded waiting) : 프로세스가 자기의 임계구역에 진입하려는 요청을 한 후부터 그 요청은 유한한 시간 이내에 허용되어야한다.
### 선점형 커널 vs 비선점형 커널
  * 선점형 커널은 프로세스가 커널 모드에서 수행되는 동안 선점되는 것을 허용한다. 따라서 경쟁조건이 생길수 있지만 비선점형 커널보다 응답이
  민첩하기 때문에 더 선호하는 커널이다.
  * 비선점형 커널은 모든 프로세스는 자발적으로 CPU의 제어를 양보하기 전까지 계속 수행된다. 따라서 경쟁조건을 염려할 필요가 없다.
## 6.3 피터슨의 해결안(Peterson's Solution)

   ![Alt text](/2.jpg)
   
   <center> <Perterson's Solution에서 프로세스 구조></center> 
   
   turn은 임계구역으로 진입할 순번을 나타낸다. 즉 turn == i이면 프로세스 Pi가 임계구역에서 실행될 수 있다.
   
   flag배열은 프로세스가 임계구역으로 진입할 준비가 되었다는것을 나타낸다. 예를 들어 flag[i]가 TRUE이면 Pi가 임계구역으로 진입할 준비가 되었다는뜻이다.
   
### 피터슨 해결책 증명
1. 상호 배제가 제대로 지켜지는가.
* flag[0] == flag[1] == true라고 해도 turn값은 i , j 둘중 하나만 가질수있다. 따라서 상호배제가 지켜진다.
2. 진행에 대한 요구조건을 만족시키는가.
3. Bounded Waiting 조건을 만족시키는가.
* Pj가 임계구역을 빠져나갈때 flag[j]의 값을 FALSE로 변경해줌으로 Pi가 진입하게 된다.
그다음 Pj는 자기자신 스스로 turn 값을 i로 바꾸어 임계구역에 들어가지 못하게 해야한다.
Pi 프로세스 임계구역에서는 turn값을 바꾸지 않기 때문이다.

## 6.4 동기화 하드웨어(Synchronization Hardware)
Perterson's Solution는 소프트웨어 기반 해결책이다. 인터럽트 되지 않은 하드웨어 명령어를 통해서 임계구역 문제를 상대적으로 간단하게 해결할수 있다.
### Test_and_Set() 명령어의 정의
   ![Alt text](/3.jpg)
   
### Test_and_set() 명령어를 사용한 상호 배제 구현
   ![Alt text](/4.jpg)
   
   만약 lock이 false라면 false를 리턴하고 lock은 true만드는 특징을 가지고 있다.
### compare_and_swap() 명령어의 정의   
   ![Alt text](/5.jpg)
   
### compare_and_swap() 명령어를 사용한 상호 배제 구현
   ![Alt text](/6.jpg)
   
   어떠한 경우에는 compare_and_swap 명령어는 value의 원래 값을 반환한다.
   
   만약 lock이 0 이라면 lock이 1로 변경되고 0을 리턴한다.
   
   만약 lock이 1이라면 1을 리턴한다.
   
   따라서 lock이 0일 경우에만 프로세스가 임계구역으로 진입할 수 있다.
   
  * 문제점
      * 위 알고리즘은 상호 배제 조건은 만족시키지만 한정된 대기 조건을 만족 시키지 못한다. Perterson's Solution 같은경우를 보면 turn이라는 변수를
      통해 제어를 하여 두 프로세스가 번갈아 실행되게 하지만 위 알고리즘은 프로세스들의 while문의 조건식이 같기 때문에 Pj가 들어갈 수 있으면
      Pi도 들어갈수 있다. 따라서 하나의 프로세스가 독점 될 가능성이 존재한다.
      
### Test_and_set() 명령어를 사용한 한정된 대기 조건을 만족시키는 상호배제
   ![Alt text](/7.jpg)
   
   boolean waiting[n] 이라는 변수를 통해 한정된 대기 조건을 만족시킨다.
   
   Pi가 임계구역에 진입하기 위해서는 waiting[i]가 FALSE 이거나 key 가 FALSE 이여야한다.
   
   waiting 변수는 다른 프로세스가 임계구역을 떠날때만 변경되며
   
   key값은 test_and_set() 명령어를 통해서만 변경된다.
   
   한 프로세스가 임계구역을 떠날때 waiting 배열을 순환하면서 다음 프로세스를 임계구역에 들어 갈 수있게 해주므로 한정된 대기시간을 만족시켜준다.






   

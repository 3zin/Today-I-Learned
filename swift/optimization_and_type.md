# Optimization and Type

![optimize](./swift_img/optimization_and_type_img/optimize.jpg)

<br></br>

## 몇 가지 사실

- Swift는 기본적으로 Value Semantics를 [장려](https://www.youtube.com/watch?v=av4i3x-aZbM) (Copy-by-Value)
- Struct, enum, tuple 등의 Value Type을 도입
- Value Type의 특징 : (1) 변수 할당시 Stack에 값 전체가 저장 (2) 전체 값이 복사되어 독립적으로 작용 (3) Heap을 쓰지 않으며 Reference Counting도 필요로 하지 않음
- identity가 아닌, 값(Value)이 중요. 이는 Equatable 프로토콜 구현을 통해 해결 가능. 

#### Value Type은 Thread간 의도하지 않은 공유로부터 안전

#### Copy는 생각보다 빠르다 

- 기본 타입(enum, struct, tuple)의 경우 상수 시간 내에 종료
- 참조 프로퍼티가 내부에 있을 경우에도(String, Array, Set, Dictionary) - Copy-on-write으로 속도 저하 보완



## Swift에서 성능에 영향을 미치는 요소

#### 1. Memory Allocation (Heap vs Stack)

- Heap 할당과 이와 관련한 관리 작업에는 꽤 많은 자원이 소비됨 (O(logn))
- 또한 그 과정이 thread-safe 해야 하기 때문에 lock등의 synchronization이 최적화를 방해
- 반면 Stack은 단순히 스택 포인터 변수값만 바꿔주는 정도 (O(1))
- 힙 할당이 루프 속에서 빈번히 일어나게 되면 성능에 영향을 미칠 수 있음



#### 2. Reference Counting (Yes vs No)

- 변수 Copy 시마다 객체에 대한 Reference Counting이 발생함
- Swift의 **ARC (Automatic Reference Counting)**
- thread-safe 해야 하기 때문에 여러 문제 발생함 (가장 대표적인 retain problem... weak/unowned)
- Atomic하게 count를 늘리고 줄여야 함



#### 3. Method Dispatch (Dynamic vs Static)

- Static Dispatch가 가능하다면, 메소드 인라이닝등을 통해 컴파일 시점에서 메소드를 처리해 콜 넘버를 줄일 수 있음
- 하지만, reference semantic에서는 다형성 문제로 인해 이와 같은 최적화가 불가능해지고, V-Table(가상 메소드 테이블, 디스패치 테이블)등을 참고해야 함. 



## Swift의 각 Type이 갖는 특성



#### 1. Struct (구조체), enum

내부에 참조 타입의 프로퍼티가 없을 경우

- Memory Allocation : **Stack**
- Reference Counting : **No**
- Method Dispatch : **Static**

내부에 참조 타입의 프로퍼티가 있을 경우

- Memory Allocation : **Stack**
- Reference Counting : **Yes** (many! - 내부에 갖고 있는 참조 프로퍼티의 수만큼 곱해짐..)
- Method Dispatch : **Static**



#### 2. Class (클래스)

- Memory Allocation : **Heap**
- Reference Counting : **Yes**
- Method Dispatch : **Dynamic**

final class의 경우

- Memory Allocation : **Heap**
- Reference Counting : **Yes**
- Method Dispatch : **Static**



#### 3. Protocol (프로토콜)

- 코드 없이 API만 정의하며, 상속 없이 다형성 구현을 가능케 해줌. 자바의 interface와 유사
- Value Type에도 적용 가능

[동적인 변수 할당이나 메소드 디스패치에 관한 자세한 내용은 이후에 프로토콜에서 따로 다룰 것](https://www.slideshare.net/YongHaYoo/ss-63881606)

- Memory Allocation : 작은 사이즈 : **Stack**, 큰 사이즈 : **Heap**
- Reference Counting : **NO**
- Method Dispatch : **Dynamic**



#### 4. Generics (제너릭스)

- 특수화가 되었다는 가정 하에 개괄적 성능은 구조체, 클래스와 동일
- 특수화를 통해서 최적화 할 수 있음. (컴파일러의 몫..)



## 정리

**Struct** : Entity 등 Value 시멘틱이 맞는 부분

**Class** : Identity가 필요한 부분. 상속과 OOP라는 강력한 패러다임

**Protocol** : 동적 다형성이 필요한 부분

**Generics** : 정적 다형성으로 가능한 부분

<br></br>

Struct 내부에 참조 프로퍼티가 많을 경우 -> Value Type으로 대체해 Reference Count 줄일 수 있음

Protocol 대상이 큰 struct면 -> class 타입의 indirect storage를 사용하거나 Copy-on-Write

Dynamic method Dispatch를 static하게 사용하기 -> final, private의 생활화. dynamic 사용과 Objc 연동 최소화. WMO 사용. 



## Reference 

https://www.slideshare.net/YongHaYoo/ss-63881606

https://academy.realm.io/kr/posts/letswift-swift-performance/


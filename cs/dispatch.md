# Dispatch

![dispatch](./cs_img/dispatch_img/dispatch.png)

- '보내다', '신속히 해결하다'
- OOP 패러다임 속에서 객체들간의 메시지 전송은 곧 객체들간의 메소드 호출. 바로 이 호출이 dispatch



## Static Dispatch

- 컴파일 타임부터 호출될 메소드가 정해져 있는 것
- 컴파일 시점에 메소드의 실제 위치를 알 수 있다면 컴파일러의 최적화인 **<u>메소드 인라이닝 (Inlining)</u>**이 진행된다 <- C++
- 효과가 있다고 판단되는 경우 컴파일러는 메소드 호출 부분에 메소드 내용을 붙여 넣어 콜 스택 오버헤드를 줄인다



```swift
struct Point{
    var x, y : CGPoint
    func draw(){
        // do nothing
    }
}

func drawPoint(param: Point){
    param.draw
}

drawPoint(point)
```

마지막 줄의 drawPoint는 drawPoint() -> Point.draw()의 두 번의 call을 하게 되지만, 인라이닝을 거치면 마지막 줄은

```swift
point.draw()
```

에서 최종적으로 

```swift
//do nothing
```

이 된다. 두 번의 콜 스택을 줄이고 최적화가 가능해진다.

<br></br>

## Dynamic Dispatch

- 호출되는 메소드가 런타임에 동적으로 정해지는 것 

```swift
class Drawable{ func draw() }

class Point : Drawable {
    var x, y: CGPoint
    override func draw { ... }
}

class Line : Drawable {
    var x1, x2, y1, y2 : CGPoint
    override func draw { ... }
}

func draw(d: drawable, withColor Color:CGColor){
    color.setFill()
    d.draw
}
```

- 여기에서 d.draw에서 호출되는 함수가 Drawable.draw인지, Point.draw인지, Line.draw인지 판단할 수 없기 때문에
- 객체의 실제 클래스 Type을 얻고, 그 클래스 타입에 속한 가상 메소드 테이블(Virtual method table, dispatch table) 을 찾아가, 실제 draw 함수의 코드 주소를 알아내 호출해야 함
- 이는 Reference Semantics의 다형성(Polymorphism)으로 인해 발생함
- 큰 문제는 없지만, **<u>컴파일러가 최적화를 못함</u>**



## Dynamic Dispatch in Objective-C

- 잘 알고 있다시피, Obj-C는 Lookup Table을 사용해 동적으로 method dispatch를 찾아 호출
- #selector(method:_) <- method signature을 이용해 찾아보는 유연한 방식
- 성능 저하 가능...



## Using Static Dispatch

- final, private
- dynamic 사용 줄이고 Objc 연동 최소화하기 *Q. How?*
- Whole Module Optimization (Xcode 지원)

 

## Reference

https://www.slideshare.net/YongHaYoo/ss-63881606

https://academy.realm.io/kr/posts/letswift-swift-performance/

http://multifrontgarden.tistory.com/133
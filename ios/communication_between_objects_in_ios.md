# Communication Between Objects in iOS

여러 작업들이 동시에 수행되는 멀티-스레딩(Multi-Threading) 상황에서 Main Queue 이외에서 발생하는 Async한 이벤트의 완료 시점을 인지하고 이를 처리하는 방법 

종속적이지 않고 ''**독립된**'' 객체들간에 ''**소통**''이 필요할 때 이와 같은 패턴들이 고안되게 됨 

iOS에서 외부 Api를 제외하고 객체들간의 (비동기적) 소통은 크게 다음 방법을 통해 이루어진다

<br>

## 1. Delegate

TableView, ScrollView등의 View가 Controller에게 말을 걸기 위해 사용되는 일반적인 방법

View는 구체적인 데이터 처리를 Controller에게 Delegate, Datasource와 같은 프로토콜을 통해 위임한다

```swift
// 1) Delegate 프로토콜 선언
protocol SomeDelegate {
    func someFunction(someProperty: Int)
}

class SomeView: UIView {
    // 2) 순환 참조를 막기 위해 weak으로 delegate 프로퍼티를 가지고 있음
    weak var delegate: SomeDelegate?
   
    func someTapped(num: Int) {
        // 3) 이벤트가 일어날 시 delegate가 동작하게끔 함
        delegate?.someFunction(someProperty: num)
    }
}
// 4) Delegate 프로토콜을 따르도록 함
class SomeController: SomeDelegate {
    var view: SomeView?
    
    init() {
        view = SomeView()
        // 6) delegate를 자신으로 설정
        view?.delegate = self
        someFunction(someProperty: 0)
    }
    
    // 5) Delegate 프로토콜에 적힌 메소드 구현
    func someFunction(someProperty: Int) {
        print(someProperty)
    }
}

let someController = SomeController()
// prints 0
```

**장점**

- 엄격한 Syntax, 필요한 메소드들이 프로토콜에 정확하게 명시됨 
- 컴파일 시 에러 발견 가능
- 로직이 비교적 따라가기 쉬움
- 모니터링용 제3 객체 필요 없음

**단점**

- 많은 코드 필요
- 많은 객체들에게 이벤트를 알려주는 것이 어려움

<br>

## 2. CallBack Closure

비동기 작업이 완료되는 시점을 인지하는 Completion Handler로서의 CallBack Closure

함수의 인자로서 클로저를 미리 전달하여 설정해 놓은 후, 완료 시점에 이를 부르도록 코딩

가장 간단하고 일반적인 방법. 사실 이하의 방법들은 이 방법의 변형이라고 보는 것이 타당...

```swift
func login(account: String, passwd: String, success: (()->Void)? = nil, fail((String)->Void)? = nil){
    //...function body...
}

login(account:account, passwd:passwd, success:{
    // success function
    }, fail:{ msg in
    // fail function
})
```

*콜백 지옥*

- 함수의 매개 변수로 넘겨지는 콜백 함수가 반복되어 코드의 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상

<br>

## 3. Notification

Observer패턴

어떤 Async한 작업이 완료되거나 특정 이벤트가 발생했을 경우 이를 디스패치 테이블 내 다른 객체(Observers)들에게 알려줌

주로 Model이 Controller에게 말을 거는 방식

**루프를 통해 해당 객체에 일어나는 변화를 지속적으로 살펴야 하기 때문에 CallBack Closure보다 자원적으로 비효율적일 수 있으나, 한 객체의 변화를 여러 객체가 관찰해야 할 경우 효율적임** 

<br>

### NSNotificaiton

NotificationCenter라는 singleton 객체를 통해서 옵저버를 등록한 객체들에게 이벤트들의 발생 여부 Notification을 post하는 방식으로 사용한다. 

메시지를 보내는 객체는 자신의 가입 객체에 대하여 알 수 없다

특정 객체의 프로퍼티와 관련되지 않은 ''일반적인 이벤트'' 처리 가능 (ex. 입력 완료와 같은 추상적인 개념)

```swift
// 1) Notification을 보내는 ViewController
class PostViewController: UIViewController {
    @IBOutlet var sendNotificationButton: UIButton!
    
    @IBAction func sendNotificationTapped(_ sender: UIButton) {
        guard let backgroundColor = view.backgroundColor else { return }
      
        // Notification에 object와 dictionary 형태의 userInfo를 같이 실어서 보낸다.
        NotificationCenter.default.post(name: Notification.Name("notification"), object: sendNotificationButton, userInfo: ["backgroundColor": backgroundColor])
    }
}

// 2) Notification을 받는 ViewController
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 옵저버를 추가해 구독이 가능하게끔 함
        NotificationCenter.default.addObserver(self, selector: #selector(notificationReceived(notification:)), name: Notification.Name("notification"), object: nil)
    }
    
    @objc func notificationReceived(notification: Notification) {
        // Notification에 담겨진 object와 userInfo를 얻어 처리 가능
        guard let notificationObject = notification.object as? UIButton else { return }
        print(notificationObject.titleLabel?.text ?? "Text is Empty")
        
        guard let notificationUserInfo = notification.userInfo as? [String: UIColor],
            let postViewBackgroundColor = notificationUserInfo["backgroundColor"] else { return }
        print(postViewBackgroundColor)
    }
}
```

장점

- 코드가 짧아 쉽게 구현이 가능.
- 다수의 객체들에게 동시에 이벤트의 발생을 알려줄 수 있음.

단점

- key 값으로 Notification의 이름과 userInfo를 서로 맞추기 때문에 컴파일 시 구독이 잘 되고 있는지 체크 불가
- Notificaiton post 이후 정보를 받을 수 없음.

<br>

### KVO(Key-Value Observing)

KVO는 객체의 프로퍼티가 변화됨을 감지할 수 있는 패턴이다. 

위의 두 패턴이 주로 Controller와 다른 객체 사이의 관계를 다룬다면 KVO 패턴은 객체와 객체 사이의 관계를 다루는데 적합하다. 

메소드나 다른 액션에서 나타나는 것이 아니라 ''구체적인 객체''의 프로퍼티의 상태에 반응하는 형태이다. 

```swift
object.addObserver(self, forKeyPath: "a"), options: [.new], context: nil))

override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
    if keyPath == a {
    	// do something
    }
  }

```

장점

- 두 객체 사이의 정보를 맞춰주는 것이 쉬움.
- new/old value를 쉽게 얻을 수 있음.
- key path로 옵저빙하기 때문에 nested objects도 옵저빙이 가능함.

단점

- 많은 value를 감지할 때는 **많은 조건문이 필요**.
- 구체적인 객체 프로퍼티만 

<br>

## 용어 : Observer ? CallBack ? Reactive?

### Callback

- 비동기 방식의 함수들이 가지는 일반적 특징. 결과값을 가지지 않으며, 처리가 끝났을 때 이어서 실행할 또 다른 함수를 입력받는 경우가 많음. 처리 시점을 예상할 수 없기 때문에 미리 로직을 전달하여 라우팅시키는 것.

### Observer

- 디자인 패턴 용어. 
- 상태변화를 감지하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴
- 주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. 발행/구독 모델로 알려져 있기도 하다

### Reactive

- 비동기적 데이터 흐름을 다루는 프로그래밍

- 명령형(imperative) 프로그래밍의 반대말. 선언형(declarative) 프로그래밍을 지향

- 함수형 프로그래밍 사용 

  - 함수를 일급 객체(first class)로 취급하여 인자, 변수 등 다양하게 활용. 람다 등의 메소드.

    > 라우팅 디자인 패턴과 관련되어 있음. 조건문의 사용을 줄이고, 함수 자체를 
    >
    > 테이블 내에 매핑하고 라우팅하여 확장성을 확보하는 것

- 데이터를 ''흐름''으로 인식하여 다룬다

= FRP(Functional Reactive Programming)



<br>

## Reference

https://brunch.co.kr/@yudong/33

https://librewiki.net/wiki/%EC%BD%9C%EB%B0%B1_%EC%A7%80%EC%98%A5

https://m.blog.naver.com/PostView.nhn?blogId=jdub7138&logNo=220937372865&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F

http://rhammer.tistory.com/95

https://medium.com/@Alpaca_iOSStudy/delegation-notification-%EA%B7%B8%EB%A6%AC%EA%B3%A0-kvo-82de909bd29

https://jdub7138.blog.me/220983291803
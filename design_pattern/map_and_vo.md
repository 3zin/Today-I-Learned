# Map and VO(Value Object) 

- 엔티티를 구체화한 객체를 전달하는 방식
- VO = DTO(Data Transfer Object, 계층간 데이터 교환을 위한 자바빈스) 

## Parameter

```swift
// swift 4.0
func sendPerson(_ name:String, _ age:Int){
    ...   
}

sendPerson("3zin", "22")
```



## Map

```swift
func sendPerson(_ param:[String : Any]){
    ...   
}

var param = [String : Any]()
param["name"] = "3zin"
param["age"] = 21
sendPerson(param)
```



## VO

```swift
func sendPerson(_ param:Person){
    ...   
}

class Person {
    var name: String
    var age: Int 
}

var param = Person()
param.name = "3zin"
param.age = 22
sendPerson(param)
```



## 분석 및 총평

- **Parameter** 방식의 경우 함수 시그니쳐 변경에 매우 취약하니 배제 

- **Map** : 선언 특성상 Value를 Any로 설정해야 하고 이는 런타임 타입 캐스팅 오류를 발생시킬 수 있음 (불필요한 작업)

  ​	   따로 설정된 putter/getter나 실제 Map이 활용되는 부분을 봐야지 어떤 객체정보를 전달하려고 하는지 알 수 있음 	   

  ​	   다만 여러 변경에 빠르게 대처할 수 있다

- **VO** :  소스의 주석이나 변수명만 봐도 전달하려는 객체 정보를 알 수 있다

  ​	 VO 자체를 바꿔야 할 경우 수정이 복잡해진다 (Map의 경우 Any이기 때문에 괜찮음)



## Reference

http://zgundam.tistory.com/65

https://okky.kr/article/196808

꼼꼼한 재은씨의 Swift 실전편 
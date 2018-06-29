# DAO (Data Access Object)

![dao](./design_pattern_img/dao_img/dao.jpg)





> 데이터 전달 객체

전체 비즈니스 로직 중에서 **데이터베이스에 SQL문을 전송하여 질의하거나 업데이트하는 로직**만 분리하여 별도의 독립적인 클래스(**DAO**)로 구현하는 디자인 패턴

- 뷰 및 비즈니스 로직 계층과 데이터베이스 처리 계층을 구분하여 명확하게 역할을 분담해 주며, 재사용을 가능하게 하고 전체 비즈니스 로직을 단순히 정리함 *(Q. DAO는 모델의 하위로 두어야 하나?)*
- 도메인 로직으로부터 데이터베이스와 직접적으로 소통하는 persistence mechanism을 숨길 수 있음
- DB에 접속하는 객체를 (테이블당)전용으로 하나만 만들고(=;= Singleton) 모든 페이지에서 이 객체를 호출해 사용하게 함으로써, DB에 접속 할때마다 커넥션을 새롭게 만들고 폐기하는 데에서 발생하는 오버헤드를 줄일 수 있음
- CRUD 메소드 제공



*DTO(VO)는 **데이터 저장 담당 클래스**를 지칭하고(순전히 getter, setter만 존재하는), DAO는 **데이터 사용기능 담당 클래스**를 지칭한다*



## Example

```swift
class DepartmentDAO {
    
    lazy var fmdb : FMDatabase! = { get } // FMDatabase property 
    
    init(){
        self.fmdb.open()
    }
    deinit {
        self.fmdb.close()
    }
    
    // CRUD 
    func create(title: String!, addr: String!) -> Bool { ... } // Create (Update)
    func find(departCd: Int = 0) -> [DepartRecord] { ... } 					   // Read
    func get(departCd: Int) -> [DepartRecord] { ... } 		   // Read
    func remove(departCd: Int) -> Bool { ... } 				   // Delete
}
```



## Reference

http://genesis8.tistory.com/214

https://zetawiki.com/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%A0%91%EA%B7%BC%EA%B0%9D%EC%B2%B4_DAO

꼼꼼한 재은씨의 Swift 실전편
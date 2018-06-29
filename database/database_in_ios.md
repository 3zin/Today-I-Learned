# Database in iOS

![android-versus-ios](./database_img/database_in_ios_img/android-versus-ios.jpg)



## Property List

- 사실상 데이터베이스는 아님
- 간단하고 짧으면서 Key-Value 형태(표준 딕셔너리)로 단순화할 수 있는 데이터의 저장을 위해 사용
- 객체 직렬화를 위한 XML 형식의 파일

- 앱의 공통 데이터나 주요 설정 정보 저장을 위해 주로 사용 

  > 마지막으로 열었던 문서, 파일에 대한 정보, 가장 많이 실행한 메뉴 정보,
  >
  > 환경 설정 데이터, 시스템 번들이나 애플리케이션 소프트웨어 정보...

- 추상화된 문자열 <string> 형태로 데이터 저장

- 누적 데이터를 관리하려면 배열 타입을 사용해야 하기 때문에 일부 데이터만 읽고 싶어도 항상 모든 데이터를 메모리에 들여와야 함. 단건의 데이터 갱신에 효율적. 

<br>

### UserDefaults

iOS에 앱을 설치하면 앱이 실행되는 시점에 데이터를 저장할 수 있는 영역인 기본 저장소가 생성됨 (Xcode에서는 확인 불가)

- 기본 저장소는 프로퍼티 리스트를 기반으로 하며 <App id>.plist 파일에 XML형식으로 데이터 저장
- iOS는 기본 저장소를 손쉽게 다룰 수 있는 UserDefaults 객체 제공 (런타임에서 동작, Singleton)

```swift
let plsit = UserDefaults.standard
plist.set(value, forKey: "name")
plist.synchronize()

var name = plist.string(forKey: "name")
```

<br>

### Custom Plist

필요에 따라서 직접 생성한 프로퍼티 파일에 데이터를 저장할 수 있음

애플리케이션의 샌드박스 내부 문서나 데이터 파일 전용 디렉터리에 저장되게 됨 (Documents)

- 공통의 데이터나 단순한 데이터라면 UserDefaults가 효율적
- 따로 그룹을 묶어서 저장해야 하거나 비슷한 형식의 데이터 그룹이 반복되는 경우에는 커스텀 Plist가 유용

- 하지만 너무 불편하다… 따라서 생략

> *샌드박스, 앱 도메인, App bundle, Documents 등 어플리케이션 구조에 관해 공부해 볼 필요가 있을 것*

<br>

## SQLite3

자세한 내용은 [database ](https://github.com/3zin/Today-I-Learned/blob/master/database/database.md)참고

- DBMS 자체는 작은 파일에 불과

- DB 파일 작성은 [*DB Browser for SQLite*](https://sqlitebrowser.org/) 같은 GUI 애플리케이션 툴을 사용하면 편함



### libsqlite3

-------------------

- Xcode에서 지원하는 SQLite 라이브러리

- 프로젝트 설정에서 libsqlite3 라이브러리를 추가하면 iOS에서 sqlite 사용 가능

**sqlite3** : DB 연결 정보가 저장되는 구조체

**sqlite3_stmt** : DB에 보낼 SQL 구문이 컴파일되어 저장되는 구조체

-> Swift에서는 둘 다 OpaquePointer 타입으로 사용 

```swift

let sql = "SQL QUERY ... "

var dbPath = "path ..."
var db: OpaquePointer? = nil
var stmt : OpaquePointer? = nil

// db를 연결. db 객체 생성
guard sqlite3_open(dbPath, &db) == SQLITE_OK else { 
 	return // error   
}

// DB 연결 종료. db 객체 해제 
defer {
   sqlite3_close(db)
}

// SQL구문을 전달할 준비. 컴파일된 SQL 구문 객체 stmt 생성
guard sqlite3_prepare(db, sql, -1, &stmt, nil) == SQLITE_OK else {
 	return // error   
}

 // 컴파일된 SQL 구문 객체 삭제. stmt 해제
defer {
    sqlite3_finalize(stmt)
}
    
// 컴파일된 SQL구문 객체를 db에 전달
if sqlite3_step(stmt) == SQLITE_DONE{
    //success!
}				 
```



### FMDB

------------

- 보다 고수준에서 데이터베이스 작업을 처리해주는 오픈소스 SQLite3 라이브러리 [fmdb](https://github.com/ccgus/fmdb)
- Objective-C로 구현, iOS와 코코아 터치 프레임워크의 특징을 반영해놓음
- libsqlite3을 재구성한 래퍼 라이브러리 



업데이트 구문은 executeUpdate 계열(Bool 반환), 쿼리 구문은 executeQuery 계열(결과 집합 반환) 메소드 사용

```swift
let db = FMDatabase(path: "file path...")

if db.open() == true {
    
    do{
        try db.executeUpdate("UPDATE SQL", values:["aaa", "bbb"]) // prepared statement
        let rs = db.exetuteQuery("QUERY SQL", values: [2]) // prepared statement
        
        while rs.next(){
            // deal with the data set using iterator pattern
        }
        
    } catch let error as NSERror {
        // error
    }
    
}
```

<br>

## Core Data





## Realm





## Reference



어플리케이션 구조 공부용 참고 링크

https://soulpark.wordpress.com/2012/09/24/ios-file-system/

http://aroundck.tistory.com/4803






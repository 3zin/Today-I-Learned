# JSON

![json](./database_img/json_img/json.PNG)





**J**ava**S**cript **O**bject **N**otation

- 서버에서 클라이언트로 데이터를 보낼 때 사용하는 양식
- XML의 가독성, 용량 문제에 대응해 간결하고 통일된 양식으로 각광을 받고 있음
- 경량(Lightweight)의 DATA-교환 형식
- 특정 언어에 종속되지 않으며, 대부분의 프로그래밍 언어에서 JSON 포맷의 데이터를 핸들링 할 수 있는 라이브러리를 제공함

<br>

## JSON의 데이터 구조

### JSON Object

![object](./database_img/json_img/object.gif)

- Key : Value

- **Unordered Set** 

- 서로 다른 여러 속성들이 위치

- Swift에서는 NSDictionary 자료형을 통해 주로 다룸

  > 다만, Swift에서 Dictionary 객체는 동일한 타임의 데이터만 저장할 수 있기 때문에 주로 Value 타입을 범용 형식 Any로 설정해 다룸. 

```json
{    
    "name": "3zin",
    "age": 22,
    "isMarried": false
}
```

<br>

### JSON Array

![array](./database_img/json_img/array.gif)

- **Ordered List**
- 비슷한 객체가 반복 나열

```json
[1,2,3,4,5]
[
    {"name": "3zin","age": 22,"isMarried": false},
    {"name": "2zin","age": 23,"isMarried": true}
    
]
```



- JSON이 지원하는 기본 자료형은 Object, Array와 함께 Number, String, Boolean, null이 있다

<br>

## 단점

- 문법 오류에 민감함
- 주석을 지원하지 않음
- 데이터 타입을 강제할 수 없음. JSON 스키마로 보완은 가능하지만 데이터 스스로 자신의 타입을 기술할 방법이 없음
- AJAX로 인해 데이터가 아닌 코드들이 JSON으로 전달될 수 있음

<br>

## Reference

https://www.json.org/json-ko.html

https://nesoy.github.io/articles/2017-02/JSON

https://namu.wiki/w/JSON
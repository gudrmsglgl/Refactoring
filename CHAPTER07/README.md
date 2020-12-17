# 캡슐화

[레코드 캡슐화하기](#id-section1)<br>


- ### 모듈을 분리하는 가장 중요한 기준
    - [x] 자신을 제외한 다른 부분에 드러내지 않아야 할 비밀을 잘 숨기느냐
    - [x] **레코드 캡슐화하기**
    - [x] **컬렉션 캡슐화하기**
    - [x] **기본형을 객체로 바꾸기**

- ### 리팩터링 시 임시변수 처리    
    - [x] **임시 변수를 질의 함수로 바꾸기**

- ### 클래스는 본래 정보를 숨기는 용도로 설계
	- [x] **여러 함수를 클래스로 묶기**
	- [x] **클래스 추출하기**
	- [x] **클래스 인라인하기**
	-  클래스 사이의 연결 관계 숨기기
		- [x] **위임 숨기기**
	- 클래스 사이의 너무 많이 숨겨서 인터페이스가 비대해질 때
		- [x] **중개자 제거하기**

- ### 함수 구현 캡슐화
	- [x] **함수 추출하기** -> **알고리즘 교체하기**

<br>
<div id='id-section1'/>

## 7.1 레코드 캡슐화하기 Encapsulate Record

```kotlin
val organization = mapOf("name" to "애크미 구스베리", "country" to "GB")
```
**🔻 레코드 캡슐화하기**
```kotlin
class Organization{
    var name: String
    var country: String
}
```
<br>

### 🔎 &nbsp;&nbsp;레코드 캡슐화할 때
- 가변 데이터를 저장하는 용도로 객체 사용
- 사용자는 무엇이 저장된 값이고 무엇이 계산된 값인지 알 필요가 없다.
- 해쉬맵은 유용하지만, 필드를 명확히 알려주지 않는 단점 <br>
	⚠️ 사용하는 곳이 많을수록 불분명함으로 인해 발생하는 문제가 커짐.<br>
	🙆‍♀️ 클래스를 사용하는 편이 낫다.
	
- 중첩된 리스트나 해시맵을 받아서 JSON 이나 XML 같은 포맷으로 직렬화할 때
	- [x] 캡슐화	


<br>

### 📍 &nbsp;&nbsp;절차
&emsp;⓵ 레코드를 담은 변수를 캡슐화한다.<br>
```
-> 레코드를 캡슐화하는 함수의 이름은 검색하기 쉽게 지어준다.
```
&emsp;⓶ 레코드를 감싼 단순한 클래스로 해당 변수의 내용을 교체한다. 이 클래스의 원본 레코드를 반환하는 접근자도 정의하고, 변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정.<br>
&emsp;⓷ 테스트한다.<br>
&emsp;⓸ 원본 레코드 대신 새로 정의한 클래스 타입의 객체를 반환하는 함수들을 새로 만든다.<br>
&emsp;⓹ 레코드를 반환하는 예전 함수를 사용하는 코드를 ⓸에서 만든 새 함수를 사용하도록 바꾼다. 필드에 접근할 때는 객체의 접근자를 사용. 한 부분을 바꿀 때마다 테스트한다.<br>
```
-> 중첩된 구조처럼 복잡한 레코드라면, 먼저 데이터를 갱신하는 클라이언트들에 주의해서 살펴본다.
   클라이언트가 데이터를 읽기만 한다면 데이터의 복제본이나 읽기전용 프락시를 반환할지 고려.
```
&emsp;⓺ 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수들을 제거.<br>
&emsp;⓻ 테스트한다.<br>
&emsp;⓼ 레코드의 필드도 데이터 구조인 중첩 구조라면 레코드 캡슐화하기와 컬렉션 캡슐화하기를 재귀적으로 적용.


<br>
<div id='id-section2'/>

## 7.2 컬렉션 캡슐화하기 Encapsulate Collection
```kotlin
class Person{
    fun course() = this._courses
    fun setCourses(list: List<>) = this._courses = list
}
```
**🔻 컬렉션 캡슐화하기**
```kotlin
class Person{
    fun course() = this._courses.slice()
    fun addCourse(course) {...}
    fun removeCourse(course) {...} 
}
```

### 🔎 &nbsp;&nbsp;컬렉션 캡슐화할 때
- 컬렉션을 소유한 클래스를 통해서만 원소를 변경하도록 함<br>
	⚠️ **게터가 컬렉션 자체를 반환하도록 하면** 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀌어버릴 수 있음<br>
	🙆‍♀️ add() 와 remove() 같은 이름의 **컬렉션 변경자 메서드를 만든다.**


<br>

### 📍 &nbsp;&nbsp;절차
&emsp;⓵ 아직 컬렉션을 캡슐화하지 않았다면 변수 캡슐화하기부터 한다.<br>
&emsp;⓶ 컬렉션에 원소를 추가/제거하는 함수를 추가한다.<br>
```
-> 컬렉션 자체를 통째로 바꾸는 세터는 제거한다. 세터를 제거할 수 없다면 인수로 받은 컬렉션을 복제해 저장하도록 한다.
```
&emsp;⓷ 정적 검사를 수행한다.<br>
&emsp;⓸ 컬렉션을 참조하는 부분을 모두 찾는다. 컬렉션의 변경자를 호출하는 코드가 모두 앞에서 추가한 추가/제거 함수를 호출하도록 수정. 수정할 때마다 테스트.<br>
&emsp;⓹ 컬렉션 게터를 수정해서 원본 내용을 수정할 수 없는 읽기전용 프락시나 복제본을 반환하게 한다.<br>
&emsp;⓺ 테스트한다.

### **ex) 수업course 목록을 필드로 지니고 있는 Person 클래스**<br>

```kotlin
class Person{ 
   private val name: String
   private var _courses: MutableList<Course>
   val courses: List<Course>
      get() = _courses
      
   constructor(name:String) {
      this.name = name
      this._courses = mutableListOf()
   }
   
   fun name() = this.name
   fun courses(list: List<Course>){
      this._courses = list
   }
}

class Course{ 
   private val name: String
   private var isAdvanced: Boolean
   
   constructor(name:String, isAdvanced) {
      this.name = name
      this.isAdvanced = isAdvanced
   }
   
   fun name() = this.name
   fun isAdvanced() = this.isAdvanced
}

// 클라이언트는 Person이 제공하는 수업 컬렉션에서 수업 정보를 얻는다.
fun numAdvancedCourses() = person.courses
   .filter{c -> c.isAdvanced()}
   .length

// 클라이언트는 세터를 이용해 수업 컬렉션을 통째로 마음대로 수정할 수 있다.
// 이런식으로 목록을 갱신하면 Person 클래스가 더는 컬렉션을 제어할 수 없어서 캡슐화가 깨진다
val basicCourseNames = readBasicCourseNames(fileName)
person.courses(
   basicCourseNames.map{name -> Course(name, false)}
)
```

#### ⚠️  필드를 참조하는 과정만 캡슐화, 필드에 담긴 내용을 캡슐화하지 않으면 컨트롤 X<br>

```kotlin
class Person{ 
  fun addCourse(course: Course) = this._courses.add(course)
  fun removeCourse(course: Course) = this._courses.remove(course)
)
```

#### ⚠️  개별 원소 추가 제거 메서드를 제공하기 때문에 setter X (제거)<br>


<br>
<div id='id-section3'/>

## 7.3 기본형을 객체로 바꾸기 Replace Primitive with Object
```kotlin
order.filter{o ->
   "high" == o.priority || "rush" == o.priority
}
```
**🔻 기본형 객체로 바꾸기**
```kotlin
order.filter{o -> o.priority.higherThan(Priority("normal"))}
```
<br>

### 🔎 &nbsp;&nbsp;기본형을 객체로 바꿀 때
- 단순한 ***출력 이상의 기능이 필요해지는 순간*** 전용 클래스를 정의
- 프로그램이 커질수록 점점 유용한 도구.
- **리팩터링 중 가장 유용한 것으로 손꼽는다.**

<br>

### 📍 &nbsp;&nbsp;절차
&emsp;⓵ 아직 변수를 캡슐화하지 않았다면 캡슐화한다.<br>
&emsp;⓶ 단순한 값 클래스(value class)를 만든다. 생성자는 기존 값을 인수로 받아서 저장하고, 이 값을 반환하는게터를 추가한다.<br>
&emsp;⓷ 정적 검사를 수행한다.<br>
&emsp;⓸ 값 클래스의 인스턴스를 새로 만들어서 필드에 저장하도록 세터를 수정한다. 이미 있다면 필드의 타입을 적절히 변경한다.<br>
&emsp;⓹ 새로 만든 클래스의 게터를 호출한 결과를 반환하도록 게터를 수정한다.<br>
&emsp;⓺ 테스트한다.<br>
&emsp;⓻ 함수 이름을 바꾸면 원본 접근자의 동작을 더 잘 드러낼 수 있는지 검토한다.<br>
```
-> 참조를 값으로 바꾸거나 값을 참조로 바꾸면 새로 만든 객체의 역할(값 또는 참조 객체)이 더 잘 들
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MzU1NjQ5NjQsLTEwOTUzMDc0OTIsOD
Q4NzEwMDAwLDE3MzM1NTE4OTAsLTE0NjM0MzUyMDQsMTQ4ODU0
Njk5OCw1MjgwMjM0MjcsLTE4MzYxODE3NjgsLTE2NjkzOTE0MD
AsODM0ODU0ODAzLC0xNTczMzc2ODddfQ==
-->
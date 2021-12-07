# 상속 다루기

[메서드 올리기](#id-section1)<br>
[필드 올리기](#id-section2)<br>
[생성자 본문 올리기](#id-section3)<br>
[메서드 내리기](#id-section4)<br>
[필드 내리기](#id-section5)<br>
[타입 코드를 서브클래스로 바꾸기](#id-section6)<br>
[서브클래스 제거하기](#id-section7)<br>
[슈퍼클래스 추출하기](#id-section8)<br>
[계층 합치기](#id-section9)<br>
[서브클래스를 위임으로 바꾸기](#id-section10)<br>

- 상속은 객체 지향 프로그래밍에서 가장 유명한 특성
- 아주 유용한 동시에 오용하기 쉽다.
- 관련 리팩터링
  - [x] 메서드 올리기
  - [x] 필드 올리기
  - [x] 생성자 본문 올리기
  - [x] 메서드 내리기
  - [x] 필드 내리기
- 계층 사이에 클래스를 추가하거나 제거하는 리팩터링
  - [x] 슈퍼클래스 추출하기
  - [x] 서브클래스 제거하기
  - [x] 계층 합치기
  - [x] 필드 값에 따라 동작이 달라지는 코드
    - 타입 코드를 서브클래스로 바꾸기
- 상속을 잘못된 곳에서 사용하거나 혹은 환경이 변해 문제가 생겼을 때
  - [x] 서브클래스를 위임으로 바꾸기
  - [x] 슈퍼클래스를 위임으로 바꾸기
  - [x] 상속을 위임으로 바꾸는 것이 핵심

<br>
<div id='id-section1'/>

## 12.1 메서드 올리기 Pull Up Method

```kotlin
   class Employee{...}

   class Salesperson : Employee {
      val name() {...}
   }

   class Engineer : Employee {
      val name() {...}
   }
```

**🔻 메서드 올리기**

```kotlin
   class Employee{
      val name() {...}
   }

   class Salesperson : Employee {...}

   class Engineer : Employee {...}
```

### 🔍 메서드 올리기

- 적용하기 가장 쉬운 상황은 메서드들의 본문 코드가 똑같을 때다.
- 테스트에 의존성이 크다 -> 차이점을 찾는 방법이 효과고 좋음
- 서로 다른 두 클래스의 두 메서드를 각각 매개변수화하면 긍정적으로 같은 메서드가 되기도 함.
- 이런 경우 가장 적은 단계를 거쳐 리팩터링하면 각각의 함수를 매개변수화한 다음 메서드를 상속 계층의 위로 올리면 된다.
- 반면 이상하고 복잡한 상황
  - [x] 해당 메서드의 본문에서 참조하는 필드들이 서브클래스에만 있는 경우.
  - [x] 이런 경우 필드들 먼저 슈퍼클래스로 올린 후에 메서드를 올리자.
- 두 메서드의 전체 흐름은 비슷하지만 -> 세부 내용이 다르다면 템플릿 메서드 만들기 고려

### 📍 절차

&emsp;⓵ 똑같이 동작하는 메서드인지 면멸히 살핀다.<br>

> 실질적으로 하는 일은 같지만 코드가 다르다면 본문 코드가 똑같이질 때까지 리팩터링.

&emsp;⓶ 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출하고 참조할 수 있는지 확인.<br>
&emsp;⓷ 메서드 시그니처가 다르다면 함수 선언 바꾸기로 슈퍼클래스에서 사용하고 싶은 형태로 통일.<br>
&emsp;⓸ 슈퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해넣는다.<br>
&emsp;⓹ 정적 검사를 수행한다.<br>
&emsp;⓺ 서브클래스 중 하나의 메서드를 제거한다.<br>
&emsp;⓻ 테스트한다.<br>
&emsp;⓼ 모든 서브클래스의 메서드가 없어질 때까지 다른 서브클래스의 메서드를 하나식 제거한다.<br>

<br>
<div id='id-section2'/>

## 12.2 필드 올리기 Pull Up Field

```kotlin
   class Employee{...}

   class Salesperson : Employee {
      private var name: String? = null
   }

   class Engineer : Employee {
      private var name: String? = null
   }
```

**🔻 필드 올리기**

```kotlin
   class Employee{
      protected var name: String? = null
   }

   class Salesperson : Employee {...}

   class Engineer : Employee {...}
```

- 서브클래스들이 독립적으로 개발되었거나 뒤늦게 하나의 계층구조로 리팩터링된 경우라면 일부 기능이 중복되어 있을 때가 있다.
- 특히 필드가 중복되기 쉽다.
- 어떤 일이 벌어지는지를 알아내려면 필드들이 어떻게 이용되는지 분석해봐야 한다.
- 분석 결과 필드들이 비슷한 방식으로 쓰인다고 판단되면 슈퍼클래스로 끌어올리자.
- 두 가지 중복을 줄일 수 있다.
  - [x] 첫째, 데이터 중복 선언을 없앨 수 있다.
  - [x] 둘째, 해당 필드를 사용하는 동작을 서브클래스에서 슈퍼클래스로 옮길 수 있다.

<br>
<div id='id-section3'/>

## 12.3 생성자 본문 올리기 Pull Up Constructor Body

```kotlin
   class Party {...}

   class Employee(
      private val name: String,
      private val id: String,
      private val monthlyCost: String) : Party() {

   }
```

**🔻 생성자 본문 올리기**

```kotlin
   class Party(private name: String) {

   }

   class Employee(
      private val name: String,
      private val id: String,
      private val monthlyCost: String) : Party(name) {...}

```

<br>
<div id='id-section4'/>

## 12.4 메서드 내리기 Push Down Method

```kotlin
   class Employee {
      fun quota(){...}
   }

   class Salesperson : Employee {...}

   class Engineer : Employee {...}
```

**🔻 메서드 내리기**

```kotlin
   class Employee {...}

   class Salesperson : Employee {...}

   class Engineer : Employee {
      fun quota(){...}
   }
```

- 특정 서브클래스 하나(혹은 소수)와만 관련된 메서드는 슈퍼클래스에서 제거하고 해당 서브클래스(들)에 추가하는 편이 깔끔.
- 해당 기능을 제공하는 서브클래스가 정확히 무엇인지를 호출자가 알고 있을 때만 적용
- 그렇지 못한 상황이라면 서브클래스에 따라 다르게 동작하는 슈퍼클래스의 기만적인 조건부 로직을 다형성으로 바꿔야 한다.

<br>
<div id='id-section5'/>

## 12.5 필드 내리기 Push Down Field

```kotlin
   class Employee {
      private val quota: String = ""
   }

   class Salesperson : Employee {...}

   class Engineer : Employee {...}
```

**🔻 필드 내리기**

```kotlin
   class Employee {...}

   class Salesperson : Employee {...}

   class Engineer : Employee {
      protected val quota: String = ""
   }
```

<br>
<div id='id-section6'/>

## 12.6 타입 코드를 서브클래스로 바꾸기 Replace Type Code with Subclasses

```kotlin
fun createEmployee(name: String, type: String){
   return Employee(name, type)
}
```

**🔻 타입 코드를 서브클래스로 바꾸기**

```kotlin
fun createEmployee(name: String, type: String){
   return when (type) {
      "engineer" -> Engineer(name)
      "salesperson" -> Salesperson(name)
      else -> Manager(name)
   }
}
```

### 🔍 타입 코드

- 타입 코드는 프로그래밍 언어에 따라 열거형이나 심볼, 문자열, 숫자 등으로 표현
- 외부 서비스가 제공하는 데이터를 다루려 할 때 딸려오는 일이 흔하다
- 타입 코드 이상(-> 서브클래스) 의 무언가가 필요할 때
- 서브 클래스 매력적인 점

  - [x] 첫째, 조건에 따라 다르게 동작하도록 해주는 다형성 제공.<br>
        동작이 달라져야 하는 함수가 여러 개일 때 특히 유용<br>
        서브클래스를 이용하면 이런 함수들에 조건부 로직을 다형성으로 바꾸기를 적용할 수 있다.
  - [x] 두 번째 매력은 특정 타입에서만 의미가 있는 값을 사용하는 필드나 메서드가 있을 때 발현된다.
    - 예를 들어 '판매 목표'는 '영업자' 유형일 때만 의미가 있다. 이런 상황이라면 서브 클래스를 만들고 필요한 서브클래스만 필드를 갖도록 정리 (필드 내리기)

- 대상 클래스에 직접 적용할지 , 타입 코드 자체에 적용할지 고민
- 전자 방식이라면 직원의 하위 타입인 엔지니어를 만들 것.
- 반면 후자는 직우너에게 직원 유형 '속성'을 부여하고, 이 속성을 클래스로 정의해 엔지니어 속성과 관리자 속성 같은 서브클래스를 만드는 식.

### 📍 절차

&emsp;⓵ 타입 코드 필드를 자가 캡슐화한다.<br>
&emsp;⓶ 타입 코드 값 하나를 선택하여 그 값에 해당하는 서브클래스를 만든다. 타입 코드 게터 메서드를 오버라이드하여 해당 타입 코드의 리터럴 값을 반환하게 한다.<br>
&emsp;⓷ 매개변수로 받은 타입 코드와 방금 만든 서브클래스를 매핑하는 선택 로직을 만든다.<br>

> 직접 상속일 때는 생성자를 팩터리 함수로 바꾸기를 적용하고 선택 로직을 팩터리에 넣는다. <br>
> 간접 상속일 때는 선택 로직을 생성자에 두면 될 것.

&emsp;⓸ 테스트한다.<br>
&emsp;⓹ 타입 코드 값 각각에 대해 서브클래스 생성과 선택 로직 추가를 반복한다. 클래스 하나가 완성될 때마다 테스트한다. <br>
&emsp;⓺ 타입 코드 필드를 제거한다. <br>
&emsp;⓻ 테스트한다.<br>
&emsp;⓼ 타입 코드 접근자를 이용하는 메서드 모두에 메서드 내리기와 조건부 로직을 다형성으로 바꾸기를 적용한다.<br>

### 예시: 직접 상속할 때

```kotlin
   // Employee 클래스
   constructor(name: String, type: String) {
      this.validateType(type)
      this._name = name
      this._type = type
   }

   fun validateType(arg) {
      if (!["engineer","manager", "salesperson"].includes(arg))
         throw Error("$arg라는 직원 유형은 없습니다.")
   }
   override fun toString() = "${this._name} (${this._type})"
```

⓵ 타입 코드 필드를 자가 캡슐화한다.<br>

```kotlin
   // Employee 클래스
   val type: String
      get() = _type

   override fun toString() = "${this._name} (${this.type})"
```

&emsp;⓶ 타입 코드 중 하나, 여기서는 엔지니어engineer를 선택. 이번에는 직접 상속 방식으로 구현. 즉, 직원 클래스 자체를 서브클래싱한다. 타입 코드 게터를 오버라이드하여 적절한 리터럴 값을 반환하기만 하면 되므로 아주 간단하게 처리할 수 있다.<br>

```kotlin
   class Engineer : Employee {
      override val type
         get() = "engineer"
   }
```

&emsp;⓷ 생성자를 팩터리 함수로 바꿔서 선택 로직을 담을 별도 장소를 마련한다.<br>

```kotlin
   fun createEmployee(name: String, type: String) {
      return Employee(name, type)
   }
```

- 새로 만든 서브클래스를 사용하기 위한 선택 로직을 팩터리에 추가한다.

```kotlin
   fun createEmployee(name, type) {
      when (type) {
	     "engineer" -> Engineer(name, type)
	     else -> Employee(name, type)
      }
   }
```

```kotlin
   class Salesperson : Employee {
      override val type
         get() = "salesperson"
   }
   class Manager : Employee {
      override val type
         get() = "manager"
   }
   fun createEmployee(name: String, type: String) {
      when (type) {
	     "engineer" -> Engineer(name, type)
	     "salesperson" -> SalesPerson(name, type)
	     "manager" -> Manager(name, type)
	     else -> Employee(name, type)
      }
   }
```

⓺ 모든 유형에 적용했다면 타입 코드 필드와 슈퍼클래스의 게터(서브클래스들에서 재정의한 메서드) 제거한다. <br>

```kotlin
    // Empolyee
    constructor(name: String, type: String) {
      this.validateType(type)
      this._name = name
      // this._type = type
   }
   /* val type
       get() = this._type */

   fun createEmployee(name: String, type: String) {
      when (type) {
	     "engineer" -> Engineer(name, type)
	     "salesperson" -> SalesPerson(name, type)
	     "manager" -> Manager(name, type)
	     else -> Error("$type라는 직업 유형은 없습니다.")
      }
   }
```

- 생성자 건네는 타입 제거

```kotlin
    // Empolyee
    constructor(name: String) {
      this.validateType(type)
      this._name = name
   }

   fun createEmployee(name: String, type: String) {
      when (type) {
	     "engineer" -> Engineer(name)
	     "salesperson" -> SalesPerson(name)
	     "manager" -> Manager(name)
	     else -> Error("$type라는 직업 유형은 없습니다.")
      }
   }
```

### 예시: 간접 상속할 때

- 직원의 서브클래스로 '아르바이트'와 '정직원'이라는 클래스가 이미 있어서 Employee를 직접 상속하는 방식으로 타입 코드 문제에 대처할 수 없다고 해보자. 직원 유형을 변경하는 기능을 유지하고 싶다는 점도 직접 상속을 사용하지 않는 이유.

```kotlin
    // Empolyee
    val type
       get() = _type
       set(value) = value

	val capitalizedType
	   get() = this._type.charAt(0).toUpperCase() + this._type.substr(1).toLowerCase()

    constructor(name: String, type: String) {
      this.validateType(type)
      this._name = name
      this._type = type
   }

   fun validateType(arg) {
      if (!["engineer","manager", "salesperson"].includes(arg))
         throw Error("$arg라는 직원 유형은 없습니다.")
   }

   override fun toString() = "${this._name} (${this.capitalizedType})"
```

⓵ 첫 번째로 할 일은 타입 코드를 객체로 바꾸기<br>

```kotlin

   class EmployeeType {
      constructor(str: String){
         this._value = str
      }
      override fun toString() = this._value
   }

   // Employee class
   val type
      get() = _type
      set(value) = value

   val typeString
      get() = this._type.toString()

	val capitalizedType
	   get() = this.typeString.charAt(0).toUpperCase() + this.typeString.substr(1).toLowerCase()

    constructor(name: String, type: String) {
      this.validateType(type)
      this._name = name
      this._type = type
   }

   fun validateType(arg) {
      if (!["engineer","manager", "salesperson"].includes(arg))
         throw Error("$arg라는 직원 유형은 없습니다.")
   }

   override fun toString() = "${this._name} (${this.capitalizedType})"
```

- 이제 바로 앞 예시와 같은 방식으로 직원 유형을 차분히 리팩터링.

```kotlin
   //Employee 클래스..
   val type
      get() = _type
      set(value) = createEmployeeType(value)

   fun createEmployeeType(str: String) = when(str) {
      "engineer" -> Engineer()
      "manager" -> Manager()
      "salesperson" -> Salesperson()
      else -> Error("$str라는 직원 유형은 없습니다."
   }

   class EmployeeType {}
   class Engineer : EmployeeType {
      override fun toString() = "engineer"
   }
   class Manager : EmployeeType {
      override fun toString() = "manager"
   }
   class Salesperson : EmployeeType {
      override fun toString() = "salesperson"
   }

```

- 이 예에서는 이름의 첫 문자만 대문자로 변환해주는 로직을 이 클래스로 옮길 수 있을 것이다.

```kotlin
   // Employee 클래스
   override fun toString() = "${this._name} (${this.type.capitalizedName})"

   // EmployeeType 클래스
   fun capitalizedName() {
      return this.toString().charAt(0).toUpperCase()
        + this.toString().subvstr(1).toLowerCase()
   }
```

<br>
<div id='id-section7'/>

## 12.7 서브클래스 제거하기 Remove Subclass

```kotlin
   class Person {
      val genderCode: String
         get() = "X"
   }
   class Male : Person {
      override val genderCode: String
         get() = "M"
   }
   class Female : Person {
      override val genderCode: String
         get() = "F"
   }
```

**🔻 서브클래스 제거하기**

```kotlin
   class Person {
      val genderCode: String
         get() = "X"
   }
```

### 🔍 서브 클래스 제거

- 서브클래싱
  - 원래 **데이터 구조와는 다른 변종**을 만들거나 종류에 따라 **동작**이 달라지게 할 수 있는 메커니즘.
- 서브 클래스를 슈퍼클래스의 필드로 대체
  - [x] 한 번도 활용되지 않을 때
  - [x] 서브클래스를 필요로 하지 않는 방식으로 만들어진 기능에서만 쓰이는 경우.
- 팩토리 패턴으로 만든 후 필드로 대체

<br>
<div id='id-section8'/>

## 12.8 슈퍼클래스 추출하기 Extract Superclass

```kotlin
   class Department {
      val totalAnnualCost() {...}
      val name() {...}
      val headCount() {...}
   }

   class Employee {
      val annualCost() {...}
      val name() {...}
      val id() {...}
   }
```

**🔻 슈퍼클래스 추출하기**

```kotlin
   class Party {
      val name() {...}
      val annualCost() {...}
   }

   class Department : Party {
      override val annualCost() {...}
      val headCount() {...}
   }

   class Employee : Party {
      val annualCost() {...}
      val id() {...}
   }
```

- 비슷한 일을 수행하는 두 클래스가 보이면 상속 메커니즘 이용
- 비슷한 부분을 슈퍼클래스로 옮겨 담을 수 있다.
  - [x] 공통된 부분이 데이터라면 필드 옮기기를 활용
  - [x] 동작이라면 메서드 올리기를 활용
- 슈퍼클래스 추출하기의 대안으로는 클래스 추출하기가 있다.
- 어느 것을 선택하느냐는 중복 동작을 상속 <-> 위임 해결

<br>
<div id='id-section9'/>

## 12.9 계층 합치기 Collapse Hierarchy

```kotlin
   class Employee {...}
   class Salesperson: Employee {...}
```

**🔻 계층 합치기**

```kotlin
   class Employee {...}
```

<br>
<div id='id-section10'/>

## 12.10 서브클래스를 위임으로 바꾸기 Replace Subclass with Delegate

```kotlin
class Order {
   val daysToShip
      get() = this._warehouse.daysToShip
}

class PriorityOrder : Order {
   val daysToShip
      get() = this._priorityPlan.daysToShip
}
```

**🔻 서브클래스를 위임으로 바꾸기**

```kotlin
class Order {
   val daysToShip
      get() = this._priorityDelegate.daysToShip ?: this._warehouse.daysToShip
}

class PriorityOrder : Order {
   val daysToShip
      get() = this._priorityPlan.daysToShip
}
```

- **🚰 속한 갈래에 따라 동작이 달라지는 객체** 들은 상속으로 표현
- 공통 데이터와 동작은 모두 슈퍼클래스에 두고 서브클래스는 자신에 맞게 기능을 추가하거나 오버라이드하면 된다.
- 🤮 상속의 단점

  - [x] 한 번만 쓸 수 있는 카드라는 것.
  - [x] 무언가가 달라져야 하는 이유가 여러 개여도 상속에서는 그중 단 하나의 이유만 선택해 기준으로 삼을 수밖에 없다.
  - [x] 예컨대 사람 객체의 동작을 '나이대'와 '소득 수준'에 따라 달리 하고 싶다면 => 서브클래스는 젊은이와 어르신이 되거나, 혹은 부자와 서민이 되어야 한다. 둘 다는 안 된다.
  - [x] 또 다른 문제로, 상속은 클래스들의 관계를 아주 긴밀하게 결합한다. => 부모를 수정하면 이미 존재하는 자식들의 기능을 헤치기가 쉽기 때문에 각별히 주의
  - [x] 그래서 자식들이 슈퍼클래스를 어떻게 상속해 쓰는지를 이해해야 한다.
  - [x] 부모와 자식이 서로 다른 모듈에 속하거나 다른 팀에서 구현한다면 문제가 더 커진다.

- 위임
  - [x] 다양한 클래스에 서로 다른 이유로 위임할 수 있다.
  - [x] 위임은 객체 사이의 **일반적인 관계**이므로 상호작용에 **필요한 인터페이스**를 명확히 정의할 수 있다.
  - [x] ✨ 즉, 상속보다 결합도가 훨씬 약하다.
  - [x] 그래서 서브클래싱(상속) 관련 문제에 직면하게 되면 흔히들 서브클래스를 위임으로 바꾸곤 한다.
  - [x] 유명한 원칙 "(클래스) 상속보다는 (객체) 컴포지션을 사용하라!"

### 🔍 절차

&emsp;⓵ 생성자를 호출하는 곳이 많다면 생성자를 팩터리 함수로 바꾼다.<br>

&emsp;⓶ 위임으로 활용할 빈 클래스를 만든다. 이 클래스의 생성자는 서브클래스의 특화된 데이터를 전부 받아야 하며, 보통은 슈퍼클래스를 가리키는 역참조도 필요하다.<br>

&emsp;⓷ 위임을 저장할 필드를 슈퍼클래스에 추가한다.<br>

&emsp;⓸ 서브클래스 생성 코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다.<br>

> 이 작업은 팩터리 함수가 수행한다. 혹은 생성자가 정확한 위임 인스턴스를 생성할 수 있는 게 확실하다면 생성자에서 수행할 수도 있다.

&emsp;⓹ 서브클래스의 메서드 중 위임 클래스로 이동할 것을 고른다.<br>

&emsp;⓺ 함수 옮기기를 적용해 위임 클래스로 옮긴다. 원래 메서드에서 위임하는 코드는 지우지 않는다.<br>

> 이 메서드가 사용하는 원소 중 위임으로 옮겨야 하는 게 있다면 함께 옮긴다. 슈퍼클래스에 유지해야 할 원소를 참조한다면 슈퍼클래스를 참조하는 필드를 위임에 추가한다.

&emsp;⓻ 서브클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브클래스의 위임 코드를 슈퍼클래스로 옮긴다. 이 때 위임이 존재하는지를 검사하는 보호 코드로 감싸야 한다. 호출하는 외부 코드가 없다면 원래 메서드는 죽은 코드가 되므로 제거한다,<br>

> 서브 클래스가 둘 이상이고 서브클래스들에서 중복이 생겨나기 시작했다면 슈퍼클래스를 추출한다. 이렇게 하여 기본 동작이 위임 슈퍼클래스로 옮겨졌다면 슈퍼클래스의 위임 메서드들에는 보호 코드가 필요 없다.

&emsp;⓼ 서브클래스의 모든 메서드가 옮겨질 때까지 5-8 과정 반복<br>

&emsp;⓽ 서브클래스들의 생성자를 호출하는 코드를 찾아서 슈퍼클래스의 생성자를 사용하도록 수정<br>

&emsp;⓾ 서브클래스를 삭제한다 (죽은 코드 제거하기)<br>

- 서브클래스를 위임으로 바꾸려 하는 이유
  - [x] 상속은 한 번만 사용할 수 있는 도구.
  - [x] 기본 예약에서 프리미엄 예약으로 동적으로 전환할 수 있도록 해야할 수 있다.
  - [x] 예컨대 aBooking.bePremium() 같은 메서드를 추가하는 식
  - [x] 완전히 새로운 객체를 만들어서 이런 상황을 피해갈 수 있는 경우도 있을 것.
  - [x] 흔한 예로, HTTP 요청을 통해 서버로부터 새로운 데이터를 받아올 수 있다.
    - [x] 처음부터 새로 만드는 방법을 사용할 수 없고, 대신 데이터 구조를 수정해야 할 때도 있다.
    - [x] 이 방식으로는 수많은 곳에서 참조되는 예약 인스턴스를 다른 것으로 교체하기 어렵다.
    - [x] 이런 상황이라면 기본 예약에서 프리미엄 예약으로 (혹은 거꾸로) 전환할 수 있게 하면 유용하다.
  - [x] 이러한 요구가 커지면 서브클래스를 위임으로 바꾸는 게 좋다.

```kotlin
--- 클라이언트(일반 예약)
booking = Booking(show, date)

--- 클라이언트(프리미엄 예약)
booking = PremiumBooking(show, date, extras)
```

&emsp;⓵ 서브클래스를 제거하려면 수정할 게 많으니 먼저 생성자를 팩터리 함수로 바꿔서 생성자 호출 부분을 캡슐화하자.<br>

```kotlin
--- 최상위...
fun createBooking(show, date): Booking {
   return Booking(show, date)
}

fun createPreminumBooking(show, date, extras): PreminumBooking {
   return PreminumBooking(show, date, extras)
}

--- 클라이언트(일반 예약)
booking = createBooking(show, date)

--- 클라이언트(프리미엄 예약)
booking = createPreminumBooking(show, date, extras)
```

- 이제 위임 클래스를 새로 만든다. 위임 클래스의 생성자는 서브클래스가 사용하던 매개변수와 <br> 예약 객체로의 역참조(back-reference)를 매개변수로 받는다.
- 역참조가 필요한 이유는 서브클래스 메서드 중 슈퍼클래스에 저장된 데이터를 사용하는 경우가 있기 때문이다. 상속에서는 쉽게 처리할 수 있지만, 위임에서는 역참조가 있어야 한다.

```kotlin
--- PreminumBookingDelegate 클래스
   constructor (hostBooking, extras) {
      this._host = hostBooking
      this._extras = extras
   }
```

&emsp;⓷⓸ 이제 새로운 위임을 예약 객체와 연결할 차례. 프리미엄 예약을 생성하는 팩터리 함수를 수정하면 된다.<br>

```kotlin
--- 최상위
fun createPremiumBooking(show, date, extras): PremiumBooking {
   val result = PremiumBooking(show, date, extras)
   result.bePremium(extras)
   return result
}

--- Booking 클래스...
fun bePremium(extras) {
   this._premiumDelegate = PremiumBookingDelegate(this, extras)
}
```

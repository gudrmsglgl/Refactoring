# 상속 다루기

[메서드 올리기](#id-section1)<br>
[필드 올리기](#id-section2)<br>
[생성자 본문 올리기](#id-section3)<br>
[메서드 내리기](#id-section4)<br>
[필드 내리기](#id-section5)<br>
[타입 코드를 서브클래스로 바꾸기](#id-section6)<br>


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
   class En
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MjI4NjcxODIsLTE5MDE5ODU0NDksLT
Y1NzkxMzk1MiwtNDUyMDc4Mjk2LC03OTM0NDk0NjMsLTIxMDIy
NDQ2ODksLTQxNDg1Mjg0Niw0Nzg1NTM0NDQsOTA1MDEzMjQ0LD
IwMTY5MDE5MTEsMTc4NDgwMDIyNiwtOTIzMDE5MzI2LDY1NzI0
OTA5OSwyNjg3OTk2MDNdfQ==
-->
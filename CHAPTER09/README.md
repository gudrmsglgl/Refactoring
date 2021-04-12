# 데이터 조직화
[변수 쪼개기](#id-section1)<br>
[필드 이름 바꾸기](#id-section2)<br>
[파생 변수를 질의 함수로 바꾸기](#id-section3)<br>
[참조를 값으로 바꾸기](#id-section4)<br>
[값을 참조로 바꾸기](#id-section5)<br>


### 📂  데이터 조직화
- 하나의 값이 여러 목적으로 사용된다면 혼란과 버그를 낳는다.
	- **변수 쪼개기**를 적용해 용도별로 분리
	- **변수 이름 바꾸기** 반드시 친해져야 함 
	- **파생 변수를 질의 함수로 바꾸기**를 활용하여 변수 자체를 완전히 없애는 것이 좋은 해법일 때 있음
	- **참조를 값으로 바꾸기** 와 **값을 참조로 바꾸기**를 사용 -> 참조, 값 구분 

<br>
<div id='id-section1'/>

## 9.1 변수 쪼개기 Split Variable

```kotlin
var temp = 2 * (height * width)
print(temp)
temp = height * width
print(temp)
```
**🔻 변수 쪼개기**
```kotlin
val perimeter = 2 * (height * width)
print(perimeter)
val area = height * width
print(area)
```
### 🔎 &nbsp;&nbsp; 변수 쪼개기 
- 대입이 두 번 이상 이뤄진다면 여러 가지 역할 수행한다는 신호 
- 역할이 둘 이상인 변수가 있다면 쪼개야 함
- 역할 하나당 변수 하나


<br>
<div id='id-section2'/>

## 9.2 필드 이름 바꾸기 Rename Field
```kotlin
class Organization {
	fun name() {...}
}
```
**🔻 필드 이름 바꾸기**
```kotlin
class Organization {
	fun title() {...}
}
```

- 구조체의 필드 이름들은 특히 더 중요
- 데이터 구조는 프로그램을 이해하는 데 큰 역할을 한다.
- 게터와 세터 이름 바꾸기도 레코드 구조체의 필드 이름 바꾸기와 똑같이 중요

<br>
<div id='id-section3'/>

## 9.3 파생 변수를 질의 함수로 바꾸기 Replace Derived Variable with Query
> 파생 변수란
> 어떤 메소드의 결과를 담기 위한 멤버 변수를 말한다. 예를 들면 누적 값을 멤버 변수로 기록해둘 수 있을 것이다. 이때 누적값이 파생 변수다. 
> 이를 질의 함수로 바꿔라. 누적값을 얻길 원한다면 질의 함수를 만들고 질의 함수가 누적 값을 계산해서 내려주도록 바꿔라. (함수형 프로그래밍)
```kotlin
fun discountedTotal() {return this._discountedTotal}
fun discount(number: Int) {
	val old = this._discount
	this._discount = number
	this._discountedToal += old - number
}
```
**🔻 파생 변수를 질의 함수로 바꾸기**
```kotlin
fun discountedTotal() {
	return this._baseTotal - this._discount
}
fun discount(number: Int) {this._discount = number}
```

### 가변 데이터 사용 시 주의사항
- 가변 데이터는 소프트웨어에 문제를 일으키는 가장 큰 골칫거리에 속함.
- 가변 데이터는 수정한 값이 연쇄 효과를 일으켜 다른 쪽 코드에 원인을 찾기 어려운 문제를 야기함.
- 가변 데이터를 완전히 배제하기한 현실적으로 불가능하지만, 가변 데이터의 유효 범위를 가능한 한 좁혀야 함.

<br>
<div id='id-section4'/>

## 9.4 참조를 값으로 바꾸기

```kotlin
class Product {
	fun applyDiscount(arg){this._price.amount -= arg}
}
```
**🔻 참조를 값으로 바꾸기**
```kotlin
class Product {
	fun applyDiscount(arg) {
		this._price = Money(this._price.amount -arg, this._price.currency)
	}
}
```

- 객체를 다른 객체에 중첩하면 내부 객체를 참조 혹은 값으로 취급할 수 있다.
- 참조냐 값이냐의 차이는 내부 **객체의 속성을 갱신하는 방식에서 드러남**
- 참조
	- 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신
- 값
	- 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다.
- 필드를 값으로 다룬다면 내부 객체의 클래스를 수정하여 값 객체로 만들 수 있다.
- 값 객체는 대체로 자유롭게 활용하기 좋음 -> 불변이기 때문
- 불변 데이터 구조는 다루기 더 쉬움
- 값 객체는 분산 시스템과 동시성 시스템에서 특히 유용

### **절차** 
- 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인
- 각각의 세터를 하나씩 제거
- 이 값 객체의 필드들을 사용하는 동시성 비교 메서드를 만든다

<br>
<div id='id-section5'/>

## 9.5 값을 참조로 바꾸기 Change Value to Reference

```kotlin
val customer = Customer(customerData)
```
**🔻 값을 참조로 바꾸기**
```kotlin
val customer = customerRepository.get(customerData.id)
```

- 주문 목록을 읽다 보면 **같은 고객이 요청한 주문이 여러 개 섞여 있을 때**
- 이 때 고객을 값으로도, 혹은 참조로도 다룰 수 있다.
- 고객 데이터를 갱신할 일이 없다면 어느 방식이든 상관없다.
- 값으로 다룬다면 고객 데이터가 각 주문에 복사되고
- 참조로 다룬다면 여러 주문이 단 하나의 데이터 구조를 참조하게 된다.
- 값을 참조로 바꾸면 엔티티 하나당 객체도 단 하나만 존재하게 되는데, 그러면 보통 이런 객체들을 한데 모아놓고 클라이언트들의 접근을 관리해주는 저장소가 필요.

### **절차** 

- 같은 부류에 속하는 객체들을 보관할 저장소를 만든다
- 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인
- 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다. 하나 수정할 때마다 테스트.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNDQzMzI5NzcsMTMyMDc2ODAsMzUyND
M0OTQ0LC0xNTgxNzI3MzExLC0xNjU4MjU3MTU4LDY3MTE2MDA2
OCwtMTQzMzU1MjczNiwxMjM5NDQwMDQ2LC0yMDY3NTIzODI5LD
YxMDA3NTksLTUyMTc4MzcyLDEyOTE0NDE2MDcsMTU2MzE4Nzc5
MSwtMTU1OTE3ODcxOCwxMDg0MTE4MTQ1LC02NzcwMzQ3MTcsLT
IwNDQ5Nzk2NjcsMTExODA2Njk2LDQ3MjI3OTMxN119
-->
# API 리팩터링

[질의 함수와 변경 함수 분리하기](#id-section1)<br>
[함수 매개변수화하기](#id-section2)<br>
[플래그 인수 제거하기](#id-section3)<br>
[객체 통째로 넘기기](#id-section4)<br>
[매개변수를 질의 함수로 바꾸기](#id-section5)<br>
[질의 함수를 매개변수로 바꾸기](#id-section6)<br>
[세터 제거하기](#id-section7)<br>
[생성자를 팩토링 함수로 바꾸기](#id-section8)<br>
[함수를 명령으로 바꾸기](#id-section9)<br>
[명령을 함수로 바꾸기](#id-section10)<br>


- ### 소프트웨어 구성 빌딩 블록 - 모듈, 함수
	- 🧩 **블록들을 끼워 맞추는 연결부 - API**
		> 이해하기 쉽고 사용하기 쉽게 만드는 일은 중요한 동시에 어려움

- ### 좋은 API
	- 데이터를 **갱신 함수**,  **조회 함수**를 명확히 구분
		> - 섞여 있다면 👉🏻 질의 함수와 변경 함수 분리하기를 적용해 갈라야함.
		> - 값 하나로 여러개로 나뉜 함수들 👉🏻 함수 매개변수화하기 적용 하나로 합침.
		> - 매개변수가 함수의 동작 모드를 전환하는 용도로만 쓰일 때 👉🏻 플래그 인수 제거하기.
   - 데이터 구조 변환
	   > - 함수 사이를 건너 다니면서 필요 이상으로 분해될 때 👉🏻 객체 통째로 넘기기 적용 
	   > - 매개변수를 질의 함수로 바꾸기 와 질의 함수를 매개변수로 바꾸기로 균형점 옮기기		
   - 클래스 대표적인 모듈
	   > - 불변 👉🏻 세터 제거하기
	   > - 호출자에 새로운 객체를 만들어 반환하려 할 때 👉🏻 생성자를 팩터리 함수로 바꾸기
	- 복잡한 함수 쪼개기
       > - 👉🏻 함수를 명령으로 바꾸기 : 함수를 객체로 변환 가능 -> 함수 추출 수월
       > - 명령 객체가 더는 필요 없어진다면 👉🏻 명령을 함수로 바꾸기
       > - 데이터가 수정됐음을 확실히 알릴 때 👉🏻 수정된 값 반환하기
       > - 오류 코드에 의존하는 과거 방식 코드 👉🏻 오류 코드를 예외로 바꾸기 
       > - 문제가 되는 조건을 함수 호출 전에 검사할 수 있다면 👉🏻 예외를 사전 확인으로 바꾸기

<br>
<div id='id-section1'/>

## 11.1 질의 함수와 변경 함수 분리하기 Separate Query from Modifier
```kotlin
fun getTotalOutstandingAndSendBill() {
   val result = customer.invoices.reduce( (total, each) -> each.amount + total, 0)
   sendBill()
   return result
}
```
**🔻 질의 함수와 변경 함수 분리하기**
```kotlin
fun totalOutstanding() {
   return customer.invoices.reduce( (total, each) -> each.amount + total, 0)
}
fun sendBill() {
   emailGateway.send(formatBill(customer))
}
```

### 👁️ 겉보기 부수효과가 없이 값을 반환해주는 함수 추구
- [x] 어느 때건 원하는 만큼 호출해도 문제가 없다
- [x] 함수의 위치를 안 어디로든 옮겨도 되며 테스트하기 쉽다
- [x] 신경 쓸 거리가 매우 적다

### 🤷🏻‍♂️ 부수효과가 있는 함수와 없는 함수 명확히 구분
- **명령---질의 분리 (command---query separation)**
	>'질의 함수 (읽기 함수)는 모두 부수효과가 없어야 한다' 규칙 따르자.
- 값을 반환하면서 부수효과도 있는 함수를 발견 
	- ⚡ **상태를 변경하는 부분과 질의하는 부분을 무조건 분리**

### 📍 절차
&emsp;⓵ 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.<br>
 > -> 함수에서 무엇을 **반환**하는지 찾는다. **변수의 이름**이 훌륭한 단초가 될 수 있음

&emsp;⓶ 새 질의 함수에서 부수효과를 모두 제거한다.<br>
&emsp;⓷ 정적 검사를 수행한다.<br>
&emsp;⓸ 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. <br>
	호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고,<br>
	원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. <br>
	하나 수정할 때마다 테스트한다.<br>
&emsp;⓹ 원래 함수에서 질의 관련 코드를 제거한다.<br>
&emsp;⓺ 테스트한다.<br>

### **ex) 이름 목록을 훑어 악당 miscreant을 찾는 함수, 악당을 찾으면 그 사람의 이름을 반환하고 경고를 울린다. 이 함수는 가장 먼저 찾은 악당만 취급한다**<br>

```kotlin
fun alertForMiscreant(people: People) {
   for (val p in people) {
      if (p == "조커") {
         setOffAlarms()
         return "조커"
      }
      if (p == "사루만") {
         setOffAlarms()
         return "사루만"
      }
   }
   return ""
}
```
&emsp;⓶ 새 질의 함수에서 부수효과를 낳는 부분 제거<br>
```kotlin
fun alertForMiscreant(people: People) {
   for (val p in people) {
      if (p == "조커") {
         //setOffAlarms() 👈🏻 제거 
         return "조커"
      }
      if (p == "사루만") {
         //setOffAlarms() 👈🏻 제거 
         return "사루만"
      }
   }
   return ""
}
```
&emsp;⓸ 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아서 새로운 질의 함수를 호출하도록 바꾸고, 이어서 변경 함수를 호출하는 코드를 아래에 삽입. <br>
```kotlin
//val found = alertForMiscreant(people) 
// 다음과 같이 바꾼다.
val found = findMiscreant(people)
alertForMisreant(people)
```
&emsp;⓹ 이제 원래의 변경 함수에서 질의 관련 코드를 없앤다.<br>
```kotlin
fun alertForMiscreant(people: People) {
   for (val p in people) {
      if (p == "조커") {
         setOffAlarms() 
         //return "조커" 👈🏻 명령함수이기 때문에 질의 코드 제거
         return
      }
      if (p == "사루만") {
         setOffAlarms() 
         //return "사루만" 👈🏻 명령함수이기 때문에 질의 코드 제거
         return
      }
   }
   //return "" 👈🏻 명령함수이기 때문에 질의 코드 제거
   return
}
```

- 더 가다듬기 **(명령함수 -> 질의 함수를 사용)** 

```kotlin
fun alertForMiscreant(people) {
   if (findMiscreant(people) != "") setOffAlarms() 
}
```


<br>
<div id='id-section2'/>

## 11.2 함수 매개변수화하기 Parameterize Function
```kotlin
fun tenPercentRaise(person: Person) {
   person.salary = person.salary.mutiply(1.1)
}

fun fivePercentRaise(person: Person) {
   person.salary = person.salary.mutiply(1.05)
}
```
**🔻 함수 매개변수화하기**
```kotlin
fun raise(person, factor) {
   person.salary = person.salary.multiply(1 + factor)
}
```

### 🔍 함수 매개변수화 
- 두 함수의 로직이 <u>**아주 비슷하고 단지 리터럴 값만 다를 때**</u>
- 다른 값만 매개변수로 처리하여 중복 없앰

### 📍 절차
&emsp;⓵ 비슷한 함수 중 하나를 선택한다.<br>
&emsp;⓶ 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다.<br>
&emsp;⓷ 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.<br>
&emsp;⓸ 테스트한다.<br>
&emsp;⓹ 매개변수로 받은 값을 사용하도록 함수 본문을 수정. 수정할 때마다 테스트<br>
&emsp;⓺ 비슷한 다른 함수를 호출하는 코드를 찾아<br>
&emsp;&emsp;&nbsp;매개변수화된 함수를 호출하도록 하나씩 수정.<br>
&emsp;&emsp;&nbsp;수정할 때마다 테스트한다.


### **ex) 대역을 다루는 세 함수의 로직**<br>

```kotlin
fun baseCharge(usage: Float) {
   if (usage < 0) return usd(0)
   val amount = 
      bottomBand(usage) * 0.03
      + middleBand(usage) * 0.05
      + topBand(usage) * 0.07
   return usd(amount)
}

fun bottomBand(usage: Float) {
   return Math.min(usage, 100)
}

fun middleBand(usage: Float) {
   return if (usage > 100) Math.min(usage, 200) - 100 else 0
}

fun topBand(usage: Float) {
   return if (usage > 200) usage - 200 else 0
}

```

&emsp;⓵ 비슷한 함수들을 매개변수화하여 통합할 때는 먼저 대상 함수 중 하나를 골라 매개변수를 추가. ( 단, 다른 함수 고려 선택 )<br>
&emsp;⓶ 리터럴을 두개 (100과 200) 사용, 그 각각은 하한과 상한을 뜻함. 함수 선언 바꾸기 적용<br>
&emsp;⓷ 리터럴들을 호출 시점에 입력하도록 바꿔보자.<br>

```kotlin
fun withinBand(    // 👈🏻 middleBand 리터럴을 매개변수로 함수 선언 바꾸기
   usage: Float,
   bottom: Float,
   top: Float
) {
   return if (usage > bottom) Math.min(usage, top) - bottom 
          else 0
}

fun baseCharge(usage: Float) {
   if (usage < 0) return usd(0)
   val amount = 
      withinBand(usage, 0, 100) * 0.03    // bottomBand(usage) * 0.03
      + withinBand(usage, 100, 200) * 0.05
      + withinBand(usage, 200, Infinity) * 0.07 // + topBand(usage) * 0.07
   return usd(amount)
}
```

<br>
<div id='id-section3'/>

## 11.3 플래그 인수 제거하기 Remove Flag Argument
```kotlin
fun setDimension(name: String, value: Float) {
   if (name == "height") {
      this._height = value
      return
   }
   if (name == "width") {
      this._width = value
      return
   }
}
```
**🔻 플래그 인수 제거**
```kotlin
fun setHeight(value: Float) { this._height = value }
fun setWidth(value: Float) { this._width = value }
```

### 플래그 인수 함수
- 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수
```kotlin
fun bookConcert(customer: Customer, isPremium: Boolean) {
   if (isPremium) {
      // 프리미엄 예약 로직
   } else {
      // 일반 예약 로직
   }
}

// 호출 시 사용되는 함수
bookConsert(customer, true)
bookConsert(customer, CustomerType.PREMIUM)
bookConsert(customer, "premium")
```
- 단점
	- [x] 호출할 수 있는 함수들이 무엇이고 어떻게 호출하는지 **🤮 이해의 어려움**
	- [x] 플래그 인수가 있으면 **🤮 함수들의 기능 차이가 잘 드러나지 않음**
	- [x] 사용할 함수를 선택한 후에도 **🤮 플래그 인수로 어떤 값을 넘겨야 하는지** 또 알아야하는 복작성
	- [x] **🤮 불리언 플래그는 코드를 읽는 이에게 뜻을 온전히 전달하지 못하기** 때문에 더욱 좋지 않음

### 📍 절차
&emsp;⓵ 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성<br>
> 주가 되는 함수에 깔끔한 분배 조건문이 포함되어 있다면 조건문 분해하기로 명시적 함수들을 생성하자. 그렇지 않으면 래핑함수 형태로 만든다.

&emsp;⓶ 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.<br>

### Ex. 배송일자를 계산하는 호출 
```kotlin
shipment.deliveryDate = deliveryDate(order, true)

// 다른 곳에서는 다음처럼 호출함
shipment.deliveryDate = deliveryDate(order, false)

// boolean 을 보면 뭘 의미하는지 의문..
// deliveryDate() 함수 코드는 다음과 같다.
fun deliveryDate(order: Order, isRush: Boolean) {
   if (isRush) {
      val deliveryTime = 0
      if (["MA", "CT"].includes(order.deliveryState)) deliveryTime = 1
      else if (["NY", "NH"].includes(order.deliveryState)) deliveryTime = 2
      else deliveryTime = 3
      return order.placeOn.plusDays(1 + deliveryTime)
   }
   else {
      val deliveryTime = 0
      if (["MA", "CT", "NY"].includes(order.deliveryState)) deliveryTime = 2
      else if (["NY", "NH"].includes(order.deliveryState)) deliveryTime = 3
      else deliveryTime = 4
      return order.placeOn.plusDays(2 + deliveryTime)
   }
}
```
**🔻 명시적인 함수를 사용해 호출자의 의도를 분명히 밝히기**
```kotlin
fun deliveryDate(order: Order, isRush: Boolean) {
   if (isRush) rushDeliveryDate(order)
   else regularDeliveryDate(order)
}

fun rushDeliveryDate(order: Order) {
   val deliveryTime = 0
   if (["MA", "CT"].includes(order.deliveryState)) deliveryTime = 1
   else if (["NY", "NH"].includes(order.deliveryState)) deliveryTime = 2
   else deliveryTime = 3
   return order.placeOn.plusDays(1 + deliveryTime)
}

fun regularDeliveryDate(order: Order) {
   val deliveryTime = 0
   if (["MA", "CT", "NY"].includes(order.deliveryState)) deliveryTime = 2
   else if (["NY", "NH"].includes(order.deliveryState)) deliveryTime = 3
   else deliveryTime = 4
   return order.placeOn.plusDays(2 + deliveryTime)
}
```


<br>
<div id='id-section4'/>

## 11.4 객체 통째로 넘기기
```kotlin
val low = room.daysTempRange.low
val high = room.daysTempRange.high
if (plan.withinRange(low, high))
```
**🔻 객체 통째로 넘기기**
```kotlin
if (plan.withinRange(room.daysTempRange))
```

### 🔍  객체 넘기기
- 객체를 통째로 넘기면 변화에 대응하기 쉽다
- 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요 없음
- 매개변수 목록이 짧아져서 함수 사용법 이해도 좋아진다
- 레코드를 통째로 넘기면 로직 중복 제거
- 🙌🏻 넘기지 말아야할 때
	- [x] 함수가 객체 자체에 의존하기를 원치 않을 때
	- [x] 객체와 함수가 서로 다른 모듈에 속한 상황
- 🧐 객체 넘기기 시 문제 발견
	- [x] 어떤 객체로부터 값 몇개를 얻은 후 그 값들만으로 무언가를 하는 로직 
		- -> 객체 안으로 집어넣어야 함을 알려주는 악취
	- [x] 한 객체가 제공하는 기중 중 항상 똑같은 일부만을 사용하는 코드가 많다면 
		- -> 그 기능만 묶어서 클래스 추출 신호
	- [x] 다른 객체의 메서드를 호출하면서 <br>
			호출하는 객체 자신이 가지고 있는 데이터 여러 개 건낼 때 <br>
			-> 객체 자신의 참조만 건네도록 수정

### 📍 절차
&emsp;⓵ 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.<br>
> -> 마지막 단계에서 이 함수의 이름을 변경해야 하니 검색하기 쉬운 이름

&emsp;⓶ 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다.<br>
&emsp;⓷ 정적 검사를 수행한다.<br>
&emsp;⓸ 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수명하며 테스트<br>
&emsp;⓹ 호출자를 모두 수정했다면 원래 함수를 인라인한다.<br>
&emsp;⓺ 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영.<br>

### Ex. 일일 최저-최고 기온이 난방 계획에서 정한 범위를 벗어나는지 확인 
```kotlin
// 출자 
val low = room.daysTempRange.low
val high = room.daysTempRange.high
if (!plan.withinRange(low, high))
   println("방 온도가 지정 범위를 벗어났습니다.")

class HeatingPlan {
   fun withinRnage(bottom, high) {
      return (bottom >= this._temperatureRange.low)
         && (top <= this._temperatureRange.high)
   }
}    
```
**🔻 통째로 넘기기**
```kotlin
class HeatingPlan {
   fun withinRnage(numberRange) {
      return (numberRange.low >= this._temperatureRange.low)
         && (numberRange.high <= this._temperatureRange.high)
   }
}    

// 호출자
if (!plan.withinRange(room.daysTempRange))
   println("방 온도가 지정 범위를 벗어났습니다.")
```

<br>
<div id='id-section5'/>

## 11.5 매개변수를 질의 함수로 바꾸기 Replace Parameter with Query
```kotlin
availableVacation(employee, employee.grade)

fun availableVacation(employee: Employee, grade: String) {
   // 연휴 계산...
}
```
**🔻 매개변수를 질의 함수로 바꾸기**
```kotlin
availableVacation(employee)

fun availableVacation(employee: Employee) {
   val grade = employee.grade
   // 연휴 계산...
}
```

### 🔍  함수에서 매개변수
- 매개변수
	- 매개변수 목록은 함수의 변동 요인
	- 함수의 동작에 변화를 줄 수 있는 일차적 수단
	- 중복은 피하는게 좋으며 짧을수록 이해하기 쉬움
- 매개변수 처리 방향
	- 피호출 함수가 '쉽게' 결정할 수 있는 값을 매개변수로 건네는 것도 일종의 중복
	- 호출하는 쪽을 간소하게 만들자.
		- [x] 책임 소재를 피호출 함수로 옮긴다. (피호출 함수가 그 역할을 수행하기에 적합할 때)
	- 제거하려는 매개변수의 값을 다른 매개변수에 질의해서(ex.object.grade) 얻을 수 있다면 질의 함수로 바꾸자
		> -> 다른 매개변수에서 얻을 수 있는 값을 별도 매개변수로 전달하는 것은 의미 없음
		
- 매개변수를 질의 함수로 바꾸지 말아야 할 상황
	- [x] 매개변수를 제거하면 피호출 함수에 원치 않는 의존성이 생길 때
	- [x] 즉, 해당 함수가 알지 못했으면 하는 프로그램 요소에 접근해야 하는 상황
	- [x] 새로운 의존생이 생기거나 or 제거하고 싶은 기존 의존성을 강화하는 경우
	- [x] -> 주로 문제의 외부 함수를 호출 or 나중에 함수 밖으로 빼내길 원하는 수용 객체에 담긴 데이터를 사용해야할 때 일어남
- 주의사항 
	- 대상 함수가 **참조 투명**해야 함
		> 참조투명: 함수에 똑같은 값을 건네 호출하면 항상 똑같이 동작한다
	- 따라서 매개변수를 없애는 대신 가변 전역 변수를 이용하는 일은 하면 안됨.	

### 📍 절차
&emsp;⓵ 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출<br>
&emsp;⓶ 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 <br>
&emsp;&emsp;&nbsp;그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다.<br> 
&emsp;&emsp;&nbsp;수정할 때마다 테스트<br>
&emsp;⓷ 함수 선언 바꾸기로 대상 매개변수를 없앤다.<br>

### Ex.  
```kotlin
// Order class
fun finalPrice() {
   val basePrice = this.quantity * this.itemPrice
   var discountLevel = 0
   if (this.quantity > 100) discountLevel = 2
   else discountLevel = 1 
   return this.discountedPrice(basePrice, discountLevel)
}

fun discountedPrice(basePrice, discountLevel) = when (descountLevel) {
   1 -> basePrice * 0.95
   2 -> basePrice * 0.9
}    
```
**🔻 매개변수 질의함수**
```kotlin
// Order class
val discountLevel = if (this.quantity > 100) 2 else 1

fun finalPrice() {
   val basePrice = this.quantity * this.itemPrice
   return this.discountedPrice(basePrice)
}

fun discountedPrice(basePrice) = when (this.descountLevel) {
   1 -> basePrice * 0.95
   2 -> basePrice * 0.9
}    
```

<br>
<div id='id-section6'/>

## 11.6 질의 함수를 매개변수로 바꾸기 Replace Query with Parameter
```kotlin
targetTemperature(plan)

fun targetTemperature(plan) {
   currentTemperature = thermostat.currentTemperature
}
```
**🔻 질의 함수를 매개변수로 바꾸기**
```kotlin
targetTemperature(plan, thermostat.currentTemperature)

fun targetTemperature(plan, thermostat.currentTemperature) {
   
}
```

### 🔍 함수 안에 참조
- 함수 안에 두기엔 거북한 참조를 발견할 때
	 - [x] 전역 변수를 참조
	 - [x] 제거하길 원하는 원소를 참조
	 - [x] 참조를 풀어내는 책임을 호출자로 옮기는 것

### 매개변수 <> 참조
- 두 사이 적절한 균형 찾아야 한다.
- 한쪽 끝은 모든 것을 매개변수로 바꿔 아주 길고 반복적인 매개변수 목록 만드는 것
- 다른 쪽 끝은 함수들끼리 많은 것을 공유하여 수많은 결합을 만들어내는 것

### 참조 투명성
- 똑같은 값을 건네면 매번 똑같은 결과를 내는 함수
- 참조 투명하지 않은 원소에 접근하는 모든 함수는 참조 투명성을 잃게 됨.
	> -> 해당 원소를 매개변수로 바꾸면 해결
	> -> 책임이 호출자로 옮겨진다 

- 장점이 크다.
	- [x] 모듈을 개발할 때 순수 함수들을 따로 구분
	- [x] 프로그램의 입출력과 기타 가변 원소들을 다루는 로직으로 순수 함수들을 겉을 감싸는 패턴 활용
	- [x] 이 리팩터링을 활용하면 프로그램의 일부를 순수 함수로 바꿀 수 있으며, 결과적으로 테스트 및 다루기 쉬워짐	
- 단점
	- [x] 어떤 값을 제공할지를 호출자가 알아내야 한다.
	- [x] 결국 호출자가 복잡 
	- [x] 책임 소재를 프로그램의 어디에 배정하느냐에 문제로 귀결

### 📍 절차
&emsp;⓵ 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리<br>
&emsp;⓶ 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출 <br>
> -> 이 함수의 이름은 나중에 수정해야 하니 검색하기 쉬운 이름으로 짓는다.

&emsp;⓷ 방금 만든 변수를 인라인하여 제거한다.<br>
&emsp;⓸ 원래 함수도 인라인한다.<br>
&emsp;⓹ 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.<br>

### Ex.  실내온도 제어 시스템. 사용자는 온도 조절기(thermostat) 온도 설정할 수 있지만, 목표 온도는 난방 계획에서 정한 범위에서만 선택할 수 있다.
```kotlin
// HeatPlan class
val targetTemperature = if (thermostat.selectedTemperature > this._max) this._max
   else if ( thermostat.selectedTemperature < this._min) this._min
   else thermostat.selectedTemperature


// 호출자
if (plan.targetTemperature > thermostat.currentTemperature) setToHeat()
else if (plan.targetTemperature < thermostat.currentTemperature) setToCool()
else setOff()
```

### 🤮 targetTemperature() 메서드가 전역 객체인 thermostat에 의존하는데 신경 쓰임 -> 전역 객체에 건네는 질의 메서드를 매개변수로 옮겨서 의존성 끊기

&emsp;⓵ 변수 추출하기를 이용하여 이 메서드에서 사용할 매개변수를 준비하는 것.
```kotlin
// HeatPlan class
val targetTemperature = 
   get() {
      val selectedTemperature = thermostat.selectedTemperature
	  return if (selectedTemperature > this._max) this._max
             else if (selectedTemperature < this._min) this._min
             else selectedTemperature   
   }
   
```
&emsp;⓶ 매개변수의 값을 구하는 코드를 제외한 나머지를 메서드로 추출하기가 수월

```kotlin
// HeatPlan class
val targetTemperature = 
   get() {
      val selectedTemperature = thermostat.selectedTemperature
	  return this.xxNEWtargetTemperature(selectedTemperature)
   }
   
fun xxNEWtargetTemperature(selectedTemperature) {
   return if (selectedTemperature > this._max) this._max
          else if (selectedTemperature < this._min) this._min
          else selectedTemperature   
}
```
&emsp;⓷ 다음으로 방금 추출한 변수를 인라인하여 원래 메서드에는 한순한 호출만 남게 된다.br>
```kotlin
// HeatPlan class
val targetTemperature = 
   get() {
	  return this.xxNEWtargetTemperature(thermostat.selectedTemperature)
   }
```
&emsp;⓸ 이어서 이 메서드까지  인라인한다.<br>
```kotlin
// 호출자
if (plan.xxNEWtargetTemperature(thermostat.selectedTemperature) > thermostat.currentTemperature) setToHeat()
else if (plan.xxNEWtargetTemperature(thermostat.selectedTemperature) < thermostat.currentTemperature) setToCool()
else setOff()
```
&emsp;⓹ 이제 새 메서드의 이름을 원래 메서드의 이름으로 바꿀 차례

```kotlin
// 호출자
if (plan.targetTemperature(thermostat.selectedTemperature) > thermostat.currentTemperature) setToHeat()
else if (plan.targetTemperature(thermostat.selectedTemperature) < thermostat.currentTemperature) setToCool()
else setOff()

// HeatPlan class
fun targetTemperature(selectedTemperature) {
	return if (selectedTemperature > this._max) this._max
           else if (selectedTemperature < this._min) this._min
           else selectedTemperature 
}
```

<br>
<div id='id-section7'/>

## 11.7 세터 제거하기 Remove Setting Method

```kotlin
class Person {
   var name: String
      get() = {...}
      set(name){
         field = name
      }
```
**🔻 세터 제거하기**

```kotlin
class Person {
   val name: String
      get() = {...}
```

### 🔍  세터 제거하기
- 세터 메서드 존재 -> 변경 및 수정될 수 있다는 뜻
- 필요한 상황
	- [x] 첫째, 사람들이 무조건 접근자 메서드를 통해서만 필드를 다루려 할 때.
	- [x] 두 번째, 클라리언트에서 생성 스크립트를 사용해 객체를 생성할 때.
		> 생성 스크립트란
		: 생성자를 호출한 후 일련의 세터를 호출하여 객체를 완성하는 형태의 코드

<br>
<div id='id-section8'/>

## 11.8 생성자를 팩터리 함수로 바꾸기 Replace Constructor with Factory Function

```kotlin
val leadEngineer = Employee(document.loadEngineer, 'E')
```
**🔻 생성자를 팩터리 함수로 바꾸기**

```kotlin
// 문자열 리터럴은 악취로 파라미터 넘겨주기 지양
val leadEngineer = createEngineer(document.loadEngineer)
```

<br>
<div id='id-section9'/>

## 11.9 함수를 명령으로 바꾸기 Replace Function with Command

```kotlin
fun score(
   candidate: Candidate,
   medicalExam: MedicalExam,
   scoringGuide: ScroingGuid
) {
   var result = 0
   var healthLevel = 0
}   
```
**🔻 함수를 명령으로 바꾸기**

```kotlin
class Scorer(
   private val candidate: Candidate,
   private val medicalExam: MedicalExam,
   private val scoringGuide: ScroingGuid
} {
   fun execute() {
      this._result = 0
      this._healthLevel = 0
   }
}
```

### 🔍  함수 -> 명령 객체 (명령)
- 🚀 **명령객체 (명령)**
	- [x] 함수만을 위한 객체 안으로 캡슐화하면 더 유용해지는 상황 
	- [x] 명령 객체는 대부분 메서드 하나로 구성, 메서드를 요청해 실행하는 것이 객체의 목적
	- [x] 평범한 함수 메커니즘 보다 훨씬 유연 -> 함수 제어 및 표현
	- [x] 되돌리기 (undo) 같은 보조 연산을 제공
	- [x] 생명주기를 더 정밀하게 제어하는 데 필요한 매개변수를 만들어주는 메서드도 제공할 수 있다.
	- [x] 상속과 훅을 이용해 사용자 맞춤형으로 만들 수도 있다.

<br>
<div id='id-section9'/>

## 11.10 명령을 함수로 바꾸기 Replace Command with Function

```kotlin
class ChargeCalculator(
   private customer: Customer,
   private usage: Usage
) {
   fun execute() {
      return this.customer.rate * this.usage
   }
}
```
**🔻 명령을 함수로 바꾸기**

```kotlin
fun charge(customer: Customer, usage: Usage) {
   return customer.rate * usage
}
```

- 명령은 그저 함수를 하나 호출해 정해진 일을 수행하는 용도로 주로 쓰인다.
- 로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크니 평범한 함수로 바꿔주는 게 낫다.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMzE5Mjk4MTIsLTUyMzAxMzQyOCwtMj
U4OTg2ODUwLC0xMDEzNzU4OTAsLTEzOTMyMzU4NDYsMjE0NzQx
Nzg1LDE3NzQ5ODEzMDksLTE2NTUwNzgzNjksMTQ5MTc0ODA3MS
wyNDYyNjYxNjQsLTM5MjE2MDIzNywxMDgwNDY3MDgyLDQ4MDc0
MTY2OCwxMzQwMjk5NzU5LC0yNzI5ODgzMTMsLTEzOTc0MjI5NT
AsLTYzODIwOTk0NiwyODg3NjY3NjQsLTE4ODEyMTg4MjcsMTAw
MjAyNjU3Nl19
-->
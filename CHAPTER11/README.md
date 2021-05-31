# API 리팩터링

[질의 함수와 변경 함수 분리하기](#id-section1)<br>
[함수 매개변수화하기](#id-section2)<br>
[플래그 인수 제거하기](#id-section3)<br>


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

// boolean 을 보
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU1NjMxNjMyNiwtOTU3Mjc2ODI0LDIwMT
c2NzIxODgsMjI2OTU1OTEsMjIxNTM0ODcsMTgzOTU3OTQwMiwx
MTkyNjk3MDE2LC0xOTczMTUzOTIyLC02MTY4MDY4MTMsNTY4MD
gyMDg0LDE0NjE1NDExNjksLTE1MjMxOTgyODQsMjg0MzE2Nzg5
LDE0NTAzODMwMjUsNDIxOTkyMjIwLDUzMzE3MzE4MSw3NjU3OT
U3NzEsMjA0ODc3NTc1NywtMTMxODQwNjY4NiwyMTM3MDMwMjU1
XX0=
-->
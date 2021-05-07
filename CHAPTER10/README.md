# 조건부 로직 간소화

[조건문 분해하기](#id-section1)<br>
[조건식 통합하기](#id-section2)<br>
[중첩 조건문을 보호 구문으로 바꾸기](#id-section3)<br>
[특이 케이스 추가하기](#id-section5)<br>


조건문의 단점
- 프로그램을 복잡하게 만드는 주요 원흉
- 복잡한 조건문
	- **조건문 분해하기**
- 논리적 조합을 명확하게 다듬을 때
	- **중첩 조건문을 보호 구문으로 바꾸기**
- 똑같은 분기 로직 (주로 switch) 여러 곳 등장 
	- **조건부 로직을 다형성으로 바꾸기**
- null 처리 로직이 거의 똑같다면 
	- **특이 케이스 추가하기**( 널 객체 추가하기 )
 
 <br>
<div id='id-section1'/>

## 10.1 조건문 분해하기 Decompose Conditional

```kotlin
if (!date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd))
   charge = quantity * plan.summerRate
else
   charge = quantity * plan.regularRate + plan.regualrServiceCharge	
```
**🔻 조건문 분해하기**
```kotlin
if (summer())
   charge = summerCharge()
else
   charge = regularCharge()	
```

- **조건문**에서 동작은 무슨 일이 일어나는지 말해주지만 **'왜' 일어나는지 제대로 말해주지 않을 때가 문제** 
- 조건문이 보이면 **조건식과 각 조건절에 의도를 살린 이름의 함수 호출**로 바꾸자.

 <br>
<div id='id-section2'/>

## 10.2 조건식 통합하기 Consolidate Conditional Expression

```kotlin
if (anEmployee.seniority < 2) return 0
if (anEmployee.monthsDisabled > 12) return 0
if (anEmployee.isPartTime) return 0
```
**🔻 조건문 통합하기**
```kotlin
if (isNotEligibleForDisability()) return 0

fun isNotEligibleForDisability() {
   return ((anEmployee.seniority < 2)
      || (anEmployee.monthsDisabled > 12)
      || (anEmployee.isPartTime))
}
```

### 🔍 조건식을 통합할 때
- **비교하는 조건은 다르**지만 그 결과로 **수행하는 동작은 같을 때**
- 조건부 코드를 통합하는 이유
	- [x] 여러 조각으로 나뉜 조건들을 통합함으로 명확해짐
	- [x] 이 작업이 함수 추출하기로 이어질 가능성이 높아짐
 - ⚡ **조건부 통합 방법**
	 - [x] or 사용하기
		 - 똑같은 결과로 이어지는 조건 검사가 순차적으로 진행
	- [x] and 사용하기
		- if문이 중첩되어 나오면 and를 사용

<br>
<div id='id-section3'/>

## 10.3 중첩 조건문을 보호 구문으로 바꾸기 Replace Nested Conditional with Guard Clauses

```kotlin
fun getPayAmount() {
   var result = 0
   if (isDead)
      result = deadAmount()
   else {
      if (isSeparated)
         result = separatedAmount()
      else {
         if (isRetried)
            result = retiredAmount()
         else 
            result = normalPayAmount()   
      }   
   }
   return result   
}
```
**🔻  중첩 조건문을 보호 구문으로 바꾸기**
```kotlin
fun getPayAmount() {
   if (isDead) return deadAmount()
   if (isSeparated) return separatedAmount()
   if (isRetried) return retiredAmount()
   return normalPayAmount()   
}
```

### 🔍 조건문의 작성 형태 
- [x] 참인 경로와 거짓인 경로 모두 정상 동작으로 어이지는 형태 
	- 👌🏻 **if 와 else 절 사용**
- [x] 한쪽만 정상인 형태
	- 👌🏻 **비정상 조건을 if에서 검사한 다음**, 조건이 참이면 **(비정상이면) 함수에서 빠져나온다.** 
	- 이와 같은 형태를 흔히 **보호 구문 (guard clause)**

###  중첩 조건문 -> 보호 구문 리팩터링
- 핵심은 의도를 부각하는 데 있다.
- if-then-else 구조를 사용할 때
	- 💬 if절과 else절에 똑같은 무게를 두어, 코드를 읽는 이에게 **양 갈래가 똑같이 중요하다는 뜻을 전달**
- 보호 구문
	- 💬 "이건 이 함수의 핵심이 아니다. **이 일이 일어나면 무언가 조치를 취한 후 함수에서 빠져나온다**" 전달.


<br>
<div id='id-section5'/>

## 10.5 특이 케이스 추가하기 Introduce Special Case

```kotlin
if (customer == "미확인 고객") customerName = "거주자"
```
**🔻  특이 케이스 추가**
```kotlin
class UnknownCustomer {
   fun name() = "거주자" 
}
```

### 🔍 특이 케이스 
- 특수한 경우의 공통 동작을 요소 하나에 모아서 사용하는 특이 케이스 패턴
- 특이 케이스를 확인하는 코드 -> 단순한 함수 호출
- 표현 방법
	- [x] 특이 케이스 객체에서 단순히 데이터를 읽기 
		> -> 반환할 값들을 담은 리터럴 객체 형태로 준비

	- [x] 그 이상의 어떤 동작을 수행해야 한다면
		> -> 필요한 메서드를 담은 객체 생성
		-> 특이 케이스 객체는 이를 캡슐화한 클래스 반환 or 
		-> 변환을 거쳐 데이터 구조에 추가시키는 형태
	
	- [x] 널은 특이 케이스로 처리해야할 때가 많음

### 📍 절차
&emsp;⓵ 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고 , false를 반환하게 한다.<br>
&emsp;⓶  특이 케이스 객체를 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 true를 반환하게 한다.<br>
&emsp;⓷ 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다.
    모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사용하도록 고친다.<br>
&emsp;⓸ 코드에 새로운 특이 케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.<br>
&emsp;⓹ 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.<br>
&emsp;⓺ 테스트한다.<br>
&emsp;⓻ 여러 함수를 클래스로 묶기나 여러 함수를 변환 함수로 묶기를 적용하여 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.<br>
&emsp;⓼ 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인한다.<br>

### Ex. 현장 site 에 인프라를 설치해 서비스 제공

```kotlin
class Site {
   fun customer() = this._customer
}

class Customer {
  fun name() {...} // 고객 이름
  fun billingPlan() {...} // 요금제
  fun setBillingPlan(arg) {...} 
  fun paymentHistory() {...} // 납부 이력
}
```

```kotlin
// 클라이언트 1...
val customer = site.customer
// ... 수많은 코드 ...
var customerName = ""
if (customer == "미확인 고객") customerName = "거주자"
else customerName = customer.name
```

```kotlin
// 클라이언트 2...
val plan = (customer == "미확인 고객") ? 
   registry.billingPlans.basic
    : customer.billingPlan
```

```kotlin
// 클라이언트 3...
if (customer != "미확인 고객") customer.billingPlan = newPlan
```

```kotlin
// 클라이언트 4...
val weeksDelinquent = (customer == "미확인 고객") ? 0 : 
	customer.paymentHistory.weeksDeliquentInLastYear
```

- 미확인 고객을 처리해야 하는 클라리언트가 여러 개 발견 
- 고객 이름으로는 "거주자", 기본 요금제(billingPlan)를 청구하고, 연체(delinquent) 기간은 0주로 분류 
- **⚡많은 곳에서 이뤄지는 이 특이 케이스 검사와 공통된 반응이⚡** 우리에게 특이 케이스 객체를 도립할 때임을 말해준다.


&emsp; **⓵ 먼저 미확인 고객인지를 나타내는 메서드를 고객 클래스에 추가<br>**
```kotlin
class Customer {
  fun isUnknown() {return false}
}
```

&emsp; **⓶ 그런 다음 미확인 고객 전용 클래스를 만든다.**
```kotlin
class UnknownCustomer {
  fun isUnknown() {return true}
}
```

&emsp; **⓷ "미확인 고객" 기대하는 곳 -> UnknownCustomer 를 반환, "미확인 고객" 인지를 검사하는 곳 모두에서 새로운 isUnknown() 메서드를 사용하도록 고쳐**

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAyMDE1NDY1MCwxMjg2ODM2MzY2LC0xOT
IzMzg4NTMsMTc0MTA5OTM4Niw0ODU1ODkyMDYsLTE5MjMwNjQ1
NTEsLTE0ODU3NjkyMTAsMTA0ODg4MjYxNiw5MjQzNTYyMzAsMT
gxMzMxNTU3NSw3ODI3Nzg3NzFdfQ==
-->
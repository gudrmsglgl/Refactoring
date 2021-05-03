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
		-> 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI0ODgyMzQ3LDEwNDg4ODI2MTYsOTI0Mz
U2MjMwLDE4MTMzMTU1NzUsNzgyNzc4NzcxXX0=
-->
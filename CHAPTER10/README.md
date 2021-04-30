# 조건부 로직 간소화

[조건문 분해하기](#id-section1)<br>
[조건식 통합하기](#id-section2)<br>


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
**🔻 조건문 분해하기**
```kotlin
if (summer())
   charge = summerCharge()
else
   charge = regularCharge()	
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY1MTA4NDMwOCwxODEzMzE1NTc1LDc4Mj
c3ODc3MV19
-->
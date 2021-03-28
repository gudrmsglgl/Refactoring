# 기능 이동

[함수 옮기기](#id-section1)<br>
[필드 옮기기](#id-section2)<br>
[문장을 함수로 옮기기](#id-section3)<br>
[문장을 호출한 곳으로 옮기기](#id-section4)<br>
[인라인 코드를 함수 호출로 바꾸기](#id-section5)<br>
[문장 슬라이드(명령-질의 원칙)](#id-section6)<br>


- 옮기기는 문장 단위
- 함수 안에서
	- [x] 바깥으로 옮길 때는 **문장을 함수로 옮기기**
	- [x] **문장을 호출한 곳으로 옮기기**로 사용
	- [x] 같은 함수 안에서 옮길 때는 **문장 슬라이드** 사용
- 한 덩어리의 문장들이 기존 함수와 같은 일을 할 때
	- [x] **인라인 코드를 함수 호출로 바꾸기**로 중복 제거
 - 반복문 리팩터링
	 - [x] **반복문 쪼개기**: 각각의 반복문이 단 하나의 일만 수행하도록 보장
	 - [x] **반복문을 파이프라인으로 바꾸기**: 반복문을 완전히 없애버림
- **죽은 코드 제거하기** 

<br>
<div id='id-section1'/>

## 8.1 함수 옮기기 Move Function
```kotlin
class Account{
   fun overdraftCharge(){...}
```
**🔻 함수 옮기기**
```kotlin
class AccountType{     👈 함수를 다른 클래스로 옮김
   fun overdraftCharge(){...}
```

### ⚙️ 모듈성 Modularity
- 좋은 소프트웨어 설계의 핵심은 모듈화가 얼마나 잘 되어 있는가.
- 모듈성이란
	- 프로그램의 어딘가를 수정하려 할 때 해당 기능과 깊이 관련된 작은 일부만 이해해도 가능하게 해주는 능력
- 모듈성 높일려면 서로 연관된 요소들을 함께 묶고, 요소 사이의 연결 관계를 쉽게 찾고 이해할 수 있도록 해야함
- 높아진 이해를 반영하려면 요소들을 이리저리 옮겨야 할 수 있다. 
- 모든 함수는 어떤 컨텍스트 안에 존재. 
- 객체 지향 프로그래밍의 **핵심 모듈화 컨텍스트는 클래스**다.
- 함수를 옮겨야 할 때
	- [x] 어떤 함수가 자신이 속한 모듈 A의 요소들보다 다른 모듈 B의 요소들을 **더 많이 참조한다면** 모듈 B로 옮기자. ( 캡슐화가 좋아짐 -> 모듈 B의 세부사항에 덜 의존하게 된다. ) 
	- [x] **호출자들의 현재 위치(호출자가 속한 모듈)나 다음 업데이트 때 바뀌리라 예상되는 위치**에 따라서도 함수를 옮겨야 할 수 있다. 
	- [x] 다른 함수 안에서 **도우미 역할로 정의된 함수 중 독립적으로 고유한 가치가 있는 것**은 접근하기 더 쉬운 장소로 옮기는 게 낫다.
	- [x] 다른 클래스로 옮겨두면 사용하기 더 편한 메소드도 있다.
- 함수를 옮길 때 유무 정하는 법
	- [x] 대상 함수의 현재 컨텍스트와 후보 컨텍스트를 둘러보면 도움이 됨.
	- [x] 대상 함수를 호출하는 함수들은 무엇인지, 대상 함수가 호출하는 함수들은 또 무엇이 있는지, 대상 함수가 사용하는 데이터는 무엇인지
	- [x] 서로 관련된 여러 함수를 묶을 때는 -> 새로운 컨텍스트 필요 -> **클래스 묶기나 추출하기로 해결.**

<br>
<div id='id-section2'/>

## 8.2 필드 옮기기 Move Field
```kotlin
class Cutomer{
   val plan get() = this._plan
   val discountRate get() = this._discountRate
}
```
**🔻 필드 옮기기**
```kotlin
class Cutomer{
   val plan get() = this._plan
   val discountRate get() = this.plan.discountRate
}
```

### 📂 올바른 데이터 구조
- 프로그램의 진짜 힘든 데이터 구조
- 경험과 도메인 주도 성계 같은 기술이 개선해줌.
- **🔨 개선해야할 때**
	- [x] 함수에 어떤 레코드를 넘길 때마다 또 다른 레코드의 필드도 함께 넘기고 있다면 
데이터 위치를 옮겨야 할 것.
	- [x] 한 레코드를 변경하려 할 때 다른 레코드의 필드까지 변경해야만 한다면 
필드의 위치가 잘못되었다는 신호

<br>
<div id='id-section3'/>

## 8.3 문장을 함수로 옮기기 Move Statements into Function
```kotlin
result.push("제목: ${person.photo.title}")
result.concat(photoData(person.photo))

fun photoData(photo: Photo) {
	return """
	| 위치: ${photo.location}
	| 날짜: ${photo.date}
	"""
}
```
**🔻 문장을 함수로 옮기기**
```kotlin
result.concat(photoData(person.photo))

fun photoData(photo: Photo) {
	return """
	| 제목: ${photo.title}
	| 위치: ${photo.location}
	| 날짜: ${photo.date}
	"""
}
```

<br>
<div id='id-section4'/>

## 8.4 문장을 호출한 곳으로 옮기기 Move Statements to Callers
```kotlin
emitPhotoData(outStream, person.photo)

fun emitPhotoData(outStream, photo) {
	outStream.write("제목: ${photo.title} ")
	outStream.write("위치: ${photo.location} ")
}
```
**🔻 문장을 호출한 곳으로 옮기기**
```kotlin
emitPhotoData(outStream, person.photo)
outStream.write("위치: ${photo.location}")

fun emitPhotoData(outStream, photo) {
	outStream.write("제목: ${photo.title} ")
}
```

<br>
<div id='id-section5'/>

## 8.5 인라인 코드를 함수 호출로 바꾸기 Replace Inline Code With Function Call
```kotlin
val appliesToMass = false
for (i in states){
	if (i == "MA") appliesToMass = true
}
```
**🔻 인라인 코드를 함수 호출**
```kotlin
appliesTomass = states.inclues("MA")
```

### ⚙️ 함수
- 여러 동작을 하나로 묶어줌
- 목적을 말하기 때문에 이해가 쉬워짐
- 중복을 없애는 데도 효과적
- 똑같은 코드 반복하는 대신 함수를 호출하면 됨

<br>
<div id='id-section6'/>

## 8.6 문장 슬라이드하기 Slide Statements
```kotlin
val pricingPlan = retrievePricingPlan()
val order = retrieveOrder()
lateinit var charge
val chargePerUnit = pricingPlan.unit
```
**🔻 문장 슬라이드하기**
```kotlin
val pricingPlan = retrievePricingPlan()
val chargePerUnit = pricingPlan.unit
val order = retrieveOrder()
lateinit var charge
```

- 문장 슬라이드하기로 코드를 한데 모아두자.
- 관련 코드끼리 모으는 작업은 다른 리팩터링의 준비 단계로 자주 행하여짐.
- 관련 있는 코드들을 명확히 구분되는 함수로 추출하는 게 그저 문장들을 한데로 모으는 것보다 나은 분리법.
- 두 가지 확인 작업
	1. 무엇을 슬라이드할지
	2. 슬라이드할 수 있는지 여부
		- [x] 부수효과가 없는지 
		- [x] 함수 추출 시 명령-질의 분리 원칙을 지키자.
			> 🔥 명령 질의 분리 원칙
			    - 명령은 어떤 데이터를 수정, 변환 등의 작업을 의미, 
			    - 질의는 어떤 데이터를 가져오는 것을 의미
			    - ref; https://webactually.com/2018/02/06/%EB%AA%85%ED%99%95%ED%95%9C-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%EB%B2%95/
			      
		- [x] 테스트해보자. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzOTMzNzkyOTgsLTcwNDI2MjY1MywtMT
czNDE1ODkyOCwtOTI5NDc2MTk1LC0xNjUwNzY4MDM2LDExOTEw
NDA1MzYsMTgyNDM2NTg1NSwtMTk1NzU5NjY2OCwtNTI1NDExMy
wzNDMwOTc1MjUsMTg2NzI2MjM3MSwxMTMxMjA0NDc2LDE2ODc3
NDgzNDYsLTUxNDE2MjgzNiwtMTYxNDM0NzU0MywxMTc1NDEwOD
k0LDE0NTQ5NDkxMzUsNTY3ODg2OTMyLDE0MTM5MDEzNSwtMTAz
NTE3MDM0MV19
-->
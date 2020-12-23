# 기능 이동

[함수 옮기기](#id-section1)<br>

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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExODI3MzA0NTUsNTY3ODg2OTMyLDE0MT
M5MDEzNSwtMTAzNTE3MDM0MSwzODYyOTY5MzQsLTEzNTQ2ODgx
NjMsLTE0ODAyNjYzODgsLTE4OTIwMTQ5MDNdfQ==
-->
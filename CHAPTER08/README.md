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


<!--stackedit_data:
eyJoaXN0b3J5IjpbNTY3ODg2OTMyLDE0MTM5MDEzNSwtMTAzNT
E3MDM0MSwzODYyOTY5MzQsLTEzNTQ2ODgxNjMsLTE0ODAyNjYz
ODgsLTE4OTIwMTQ5MDNdfQ==
-->
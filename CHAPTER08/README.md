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
class AccountType{
   fun overdraftCharge(){...}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMTkwNDk4NzcsMTQxMzkwMTM1LC0xMD
M1MTcwMzQxLDM4NjI5NjkzNCwtMTM1NDY4ODE2MywtMTQ4MDI2
NjM4OCwtMTg5MjAxNDkwM119
-->
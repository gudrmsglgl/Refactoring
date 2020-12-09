# 기본적인 리팩터링
[함수 추출하기](#id-section1)<br>


### 저수준 리팩터링
- **추출**은 결국 이름 짓기이며, 코드 이해도가 높아지다 보면 이름을 바꿔야 할 때가 많다.
- **함수 선언 바꾸기** 
    - 함수의 이름을 변경할 때 많이 쓰인다.
    - 함수의 인수를 추가 제거할 때 적용
- **함수 이름 바꾸기**
    - 바꿀 대상이 변수일 때 
    - 변수 캡슐화하기와 관련 깊다.
- **매개변수 객체 만들기**
    - 자주 함께 뭉쳐 다니는 인수 -> 객체 하나로 묶기

### 고수준 리팩터링
- 함수를 만들고 나면 다시 고수준 모듈로 묶어야 한다.
- 함수를 그룹으로 묶을 때는 **여러 함수를 클래스로 묶기**를 이용
- 읽기 전용 데이터
    - 여러 함수를 변환 함수로 묶기 
- 단계 쪼개기
    - 한데 묶은 모듈들의 작업 처리 과정을 명확한 단계로 구분 짓기    

<br>
<div id='id-section1'/>

## 6.1 함수 추출하기

```kotlin
fun printOwing(val invoice){
	printBanner()
	val outstanding = calculateOutstanding()
	
	// 세부 사항 출력 
	println("고객명: ${invoice.customer}")
	println("채무액: ${outstanding}")
}
```

```kotlin
fun printOwing(val invoice){
	printBanner()
	val outstanding = calculateOutstanding()
	
	// 세부 사항 출력 
	println("고객명: ${invoice.customer}")
	println("채무액: ${outstanding}")
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY3Mjk2NDg1XX0=
-->
# API 리팩터링

[질의 함수와 변경 함수 분리하기](#id-section1)<br>


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
 > -> 함수에서 무엇을 반환하는지 찾는다. 변수의 이름이 훌륭한 단초가 될 수 있음

&emsp;⓶  특이 케이스 객체를 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 true를 반환하게 한다.<br>
&emsp;⓷ 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다.
    모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사용하도록 고친다.<br>
&emsp;⓸ 코드에 새로운 특이 케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.<br>
&emsp;⓹ 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.<br>
&emsp;⓺ 테스트한다.<br>
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMTM3MzQ0MjksNTMzMTczMTgxLDc2NT
c5NTc3MSwyMDQ4Nzc1NzU3LC0xMzE4NDA2Njg2LDIxMzcwMzAy
NTUsLTIxNDEzNjg2NzcsMTYxMTQxNDk1MCwtMzY1MjU2MTA4XX
0=
-->
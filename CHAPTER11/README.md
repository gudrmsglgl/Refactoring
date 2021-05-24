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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MjMxOTgyODQsMjg0MzE2Nzg5LDE0NT
AzODMwMjUsNDIxOTkyMjIwLDUzMzE3MzE4MSw3NjU3OTU3NzEs
MjA0ODc3NTc1NywtMTMxODQwNjY4NiwyMTM3MDMwMjU1LC0yMT
QxMzY4Njc3LDE2MTE0MTQ5NTAsLTM2NTI1NjEwOF19
-->
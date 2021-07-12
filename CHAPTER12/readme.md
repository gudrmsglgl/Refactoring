# 상속 다루기

[메서드 올리기](#id-section1)<br>


- 상속은 객체 지향 프로그래밍에서 가장 유명한 특성
- 아주 유용한 동시에 오용하기 쉽다.
- 관련 리팩터링
	- [x] 메서드 올리기
	- [x] 필드 올리기
	- [x] 생성자 본문 올리기
	- [x] 메서드 내리기
	- [x] 필드 내리기
- 계층 사이에 클래스를 추가하거나 제거하는 리팩터링
	- [x] 슈퍼클래스 추출하기
	- [x] 서브클래스 제거하기
	- [x] 계층 합치기
	- [x] 필드 값에 따라 동작이 달라지는 코드 
		- 타입 코드를 서브클래스로 바꾸기
- 상속을 잘못된 곳에서 사용하거나 혹은 환경이 변해 문제가 생겼을 때
	- [x] 서브클래스를 위임으로 바꾸기
	- [x] 슈퍼클래스를 위임으로 바꾸기
	- [x] 상속을 위임으로 바꾸는 것이 핵심


<br>
<div id='id-section1'/>

## 12.1 메서드 올리기 Pull Up Method

```kotlin
   class Employee{...}

   class Salesperson : Employee {
      val name() {...}
   }

   class Engineer : Employee {
      val name() {...}
   }
```
**🔻 메서드 올리기**

```kotlin
   class Employee{
      val name() {...}
   }

   class Salesperson : Employee {...}

   class Engineer : Employee {...}
```

### 🔍 메서드 올리기
- 적용하기 가장 쉬운 상황은 메서드들의 본문 코드가 똑같을 때다.
- 테스트에 의존성이 크다 -> 차이점을 찾는 방법이 효과고 좋음
- 서로 다른 두 클래스의 두 메서드를 각각 매개변수화하면 긍정적으로 같은 메서드가 되기도 함.
- 이런 경우 가장 적은 단계를 거쳐 리팩터링하면 각각의 함수를 매개변수화한 다음 메서드를 상속 계층의 위로 올리면 된다.
- 반면 이상하고 복잡한 상황
	- [x] 해당 메서드의 본문에서 참조하는 필드들이 서브클래스에만 있는 경우.
	- [x] 이런 경우 필드들 먼저 슈퍼클래스로 올린 후에 메서드를 올리자.
- 두 메서드의 전체 흐름은 비슷하지만 -> 세부 내용이 다르다면 템플릿 메서드 만들기 고려


### 📍 절차
&emsp;⓵ 똑같이 동작하는 메서드인지 면멸히 살핀다.<br>
> 실질적으로 하는 일은 같지만 코드가 다르다면 본문 코드가 똑같이질 때까지 리팩터링.

&emsp;⓶ 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출하고 참조할 수 있는지 확인.<br>
&emsp;⓷ 메서드 시그니처가 다르다면 함수 선언 바꾸기로 슈퍼클래스에서 사용하고 싶은 형태로 통일.<br>
&emsp;⓸ 슈퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해넣는다.<br>
&emsp;⓹ 정적 검사를 수행한다.<br>
&emsp;⓺ 서브클래스 중 하나의 메서드를 제거한다.<br>
&emsp;⓻ 테스트한다.<br>
&emsp;⓼ 모든 서브클래스의 메서드가 없어질 때까지 다른 서브클래스의 메서드를 하나식 제거한다.<br>



<br>
<div id='id-section2'/>

## 12.2 필드 올리기 Pull Up Field
```kotlin
   class Employee{...}

   class Salesperson : Employee {
      private var name: String? = null
   }

   class Engineer : Employee {
      private var name: String? = null
   }
```
**🔻 필드 올리기**

```kotlin
   class Employee{
      protected var name: String? = null
   }

   class Salesperson : Employee {...}

   class Engineer : Employee {...}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDMwMDgwNjQsLTQxNDg1Mjg0Niw0Nz
g1NTM0NDQsOTA1MDEzMjQ0LDIwMTY5MDE5MTEsMTc4NDgwMDIy
NiwtOTIzMDE5MzI2LDY1NzI0OTA5OSwyNjg3OTk2MDNdfQ==
-->
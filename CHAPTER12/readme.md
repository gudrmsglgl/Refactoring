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
- 이런 경우 가장 적은 단계를 거쳐 리팩터링하면 기
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNTY0NjQwLDIwMTY5MDE5MTEsMTc4ND
gwMDIyNiwtOTIzMDE5MzI2LDY1NzI0OTA5OSwyNjg3OTk2MDNd
fQ==
-->
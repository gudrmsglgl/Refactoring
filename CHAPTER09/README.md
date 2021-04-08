# 데이터 조직화
[변수 쪼개기](#id-section1)<br>
[필드 이름 바꾸기](#id-section2)<br>



### 📂  데이터 조직화
- 하나의 값이 여러 목적으로 사용된다면 혼란과 버그를 낳는다.
	- **변수 쪼개기**를 적용해 용도별로 분리
	- **변수 이름 바꾸기** 반드시 친해져야 함 
	- **파생 변수를 질의 함수로 바꾸기**를 활용하여 변수 자체를 완전히 없애는 것이 좋은 해법일 때 있음
	- **참조를 값으로 바꾸기** 와 **값을 참조로 바꾸기**를 사용 -> 참조, 값 구분 

<br>
<div id='id-section1'/>

## 9.1 변수 쪼개기 Split Variable

```kotlin
var temp = 2 * (height * width)
print(temp)
temp = height * width
print(temp)
```
**🔻 변수 쪼개기**
```kotlin
val perimeter = 2 * (height * width)
print(perimeter)
val area = height * width
print(area)
```
### 🔎 &nbsp;&nbsp; 변수 쪼개기 
- 대입이 두 번 이상 이뤄진다면 여러 가지 역할 수행한다는 신호 
- 역할이 둘 이상인 변수가 있다면 쪼개야 함
- 역할 하나당 변수 하나


<br>
<div id='id-section2'/>

## 9.2 필드 이름 바꾸기 Rename Field
```kotlin
class Organization {
	fun name() {...}
}
```
**🔻 필드 이름 바꾸기**
```kotlin
class Organization {
	fun title() {...}
}
```

- 구조체의 필드 이름들은 특히 더 중요
- 데이터 구조는 프로그램을 이해하는 데 큰 역할을 한다.
- 게터와 세터 이름 바꾸기도 레코드 구조체의 필드 이름 바꾸기와 똑같이 중요
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY3NzAzNDcxNywtMjA0NDk3OTY2NywxMT
E4MDY2OTYsNDcyMjc5MzE3XX0=
-->
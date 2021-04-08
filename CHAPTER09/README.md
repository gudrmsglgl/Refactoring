# 데이터 조직화
[변수 쪼개기](#id-section1)<br>



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


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg1MDczNTM5MiwtMjA0NDk3OTY2NywxMT
E4MDY2OTYsNDcyMjc5MzE3XX0=
-->
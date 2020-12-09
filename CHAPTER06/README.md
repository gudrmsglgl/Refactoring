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
fun printOwing(invoice: Invoice){
	printBanner()
	val outstanding = calculateOutstanding()
	
	// 세부 사항 출력 
	println("고객명: ${invoice.customer}")
	println("채무액: ${outstanding}")
}
```
**🔻&nbsp;&nbsp;함수 추출 후**
```kotlin
fun printOwing(invoice: Invoice){
	printBanner()
	val outstanding = calculateOutstanding()
	printDetail(invoice, outstanding)
}

fun printDetail(invoice: Invoice, outstanding: Int){
	println("고객명: ${invoice.customer}")
	println("채무액: ${outstanding}")
}
```

### 🔎 &nbsp;&nbsp;코드를 독립된 함수로 묶어야 할 때
 &nbsp;&nbsp;&nbsp;&nbsp;📌&nbsp;&nbsp;**목적과 구현을 분리**
```
  * 코드를 보고 무슨 일을 하는지 파악하는 데 한참이 걸린다면 함수로 추출
   '무슨 일'에 걸맞는 이름을 짓는다.
  😊 함수를 아주 짧게, 단 몇 줄만 담도록 작성하는 습관 
     💭 함수 호출이 많아져서 성능이 느려질까 걱정하지 말아라
  😫 5~6줄 이상 냄새 풍김
```
- 길이를 기준
 : 한 화면을 넘어가면 안 된다
- 재사용성 기준
: 두 번 이상 사용될 코드는 함수로 만들고, 한 번만 쓰이는 코드는 인라인 상태로 놔두는 것.
<br>

### 📍 &nbsp;&nbsp;함수 추출 절차
① 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다 ('어떻게' 아닌 '무엇을' 하는지가 드러나게)
```
* 대상 코드가 매우 간던하더라도 함수로 뽑아서 목적이 더 잘 드러나는 이름을 붙일 수 있다면 추출한다.
* 이름이 떠오르지 않는다면 함수로 추출하면 안 되는 신호.
* 추출하는 과정에서 좋은 이름이 떠오를 수도 있으니 처음부터 최선의 이름부터 짓고 시작할 필요 없다.
* 함수로 추출해서 사용해보고 효과가 크지 않다면 다시 원래 상태로 인라인해도 된다.
```
② 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여넣는다.
③ 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.

 ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIwNDk2Mjk1MCwtMTEwMDkwMjc5MSwtMT
Y2MDU5NjgxMCwtMTQ4NTUzODIwNV19
-->
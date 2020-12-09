# 기본적인 리팩터링
[함수 추출하기](#id-section1)<br>
[함수 인라인하기](#id-section2)<br>
[변수 추출하기](#id-section3)<br>


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

## 6.1 함수 추출하기 Extract Function

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
<br>

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

### 📍 &nbsp;&nbsp;절차
① 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다 ('어떻게' 아닌 '무엇을' 하는지가 드러나게)
```
* 대상 코드가 매우 간던하더라도 함수로 뽑아서 목적이 더 잘 드러나는 이름을 붙일 수 있다면 추출한다.
* 이름이 떠오르지 않는다면 함수로 추출하면 안 되는 신호.
* 추출하는 과정에서 좋은 이름이 떠오를 수도 있으니 처음부터 최선의 이름부터 짓고 시작할 필요 없다.
* 함수로 추출해서 사용해보고 효과가 크지 않다면 다시 원래 상태로 인라인해도 된다.
```
② 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여넣는다.<br>
③ 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
```
-> 함수에는 지역 변수와 매개변수가 있기 마련. 
   가장 일반적인 처리 방법은 이런 변수 모두를 인수로 전달.
-> 추출한 코드에서만 사용하는 변수가 추출한 함수 밖에 선언되어 있다면 추출한 함수 안에서 선언 하도록 수정
-> 추출한 코드 안에서 값이 바뀌는 변수 중에서 값으로 전달되는 것들은 주의해서 처리.
   이런 변수가 하나뿐이라면 추출한 코드를 질의 함수로 취급해서 그 결과(반환 값)을 해당 변수에 대입
-> 때로는 추출한 코드에서 값을 수정하는 지역 변수가 너무 많을 수 있다. 
   이럴 때는 함수 추출을 멈추고, 변수 쪼개기가 임시 변수를 질의 함수로 바꾸기와 같은 다른 리팩터링 적용해서 변수를 사용하는 코드를 단순하게 바꿔본다. 그런 다음 함수 추출을 다시 시도.      
```
 ④ 변수를 다 처리했다면 컴파일한다.<br>
 ⑤ 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다 (즉, 추출한 함수로 일을 위임)<br>
 ⑥ 테스트한다.<br>
 ⑦ 다른 코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다. 있다면 방금 추출한 새 함수를 호출하도록 바꿀지 검토한다. (인라인 코드를 함수 호출로 바꾸기)

<br>
<div id='id-section2'/>

## 6.2 함수 인라인하기 Inline Function
```kotlin
fun getRating(driver: Driver): Int{
	retrun moreThanFiveLateDeliveries(driver) ? 2: 1
}

fun moreThanFiveLateDeliveries(driver: Driver): Int {
	return driver.numberOfLateDeliveries > 5
}
```
**🔻&nbsp;&nbsp;함수 인라인**
```kotlin
fun getRating(driver: Driver): Int{
	retrun (driver.numberOfLateDeliveries > 5) ? 2: 1
}
```
<br>

### 🔎 &nbsp;&nbsp;함수를 인라인 할 때 
- 함수 본문이 이름만큼 명확한 경우
- 함수 본문 코드를 이름만큼 깔끔하게 리팩터링할 때
- 리팩터링 과정에서 잘못 추출된 함수들도 다시 인라인
- 간접 호출을 너무 과하게 쓰는 코드
- 다른 함수로 단순히 위임하기만 하는 함수들이 너무 많아서 위임 관계가 복잡하게 얽힌 경우

<br>

### 📍 &nbsp;&nbsp;절차
① 다형 메서드(polymorphic method)인지 확인.
```
-> 서브클래스에서 오버라이드하는 메서드는 인라인하면 안 된다.
```
② 인라인할 함수를 호출하는 곳을 모두 찾는다.<br>
③ 각 호출문을 함수 본문으로 교체한다.<br>
④ 하나씩 교체할 때마다 테스트한다.<br>
```
-> 인라인 작업을 한 번에 처리할 필요는 없다.
인라인하기가 까다로운 부분이 있다면 일단 남겨두고 여유가 생길 때마다 틈틈이 처리한다.
```
⑤ 함수 정의(원래 함수)를 삭제한다.

<br>
<div id='id-section3'/>

## 6.3 변수 추출하기 Extract Variable
```kotlin
return order.quantity * order.itemPrice -
	Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
	Math.min(order.quantity * order.itemPrice * 0.1, 100)
```
**🔻&nbsp;&nbsp;변수 추출**
```kotlin
val basePrice = order.quantity * order.itemPrice
val quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05
val shipping = Math.min(basePrice * 0.1, 100)
return basePrice - quantityDiscount + shipping
```

### 🔎 &nbsp;&nbsp;변수 추출할 때 
- 표현식이 너무 복잡한 경우
	- 이럴 때 지역 변수 활용 
	- 코드의 목적 훨씬 명확하게 드러낼 수 있음
	- 디버깅에도 도움이 됨. (중단점 지정, 상태 출력 문장 추가 가능)
- 변수 추출은 곧 이름을 붙이고 싶다.
	- 이름이 들어갈 문맥을 살피자.
	- 현재 함수 안에서만 의미가 있다면 변수로 추출
	- ~~함수를 벗어난~~ **넓은 문맥**에서까지 의미가 된다면 그 **넓은 범위에서 통용**되는 이름
		- 변수가 아닌 (주로) 함수로 추출해야 한다.
		- 이름이 통용되는 문맥을 넓히면 다른 코드에서 사용할 수 있기 때문에 같은 표현식을 중복해서 작성하지 않아도 됨
		- 중복이 적으면서 의도가 잘 드러나는 코드를 작성 
```kotlin
class Order{
	val data
	val quantity
		get() = this.data.quantity
	val itemPrice
		get() = this.data.itemPrice
	val price
		get() = this.quantity * this.itemPrice - 
		Math.max(0, this.quantity - 500) * this.itemPrice * 0.05 +
		Math.min(this.quantity * this.itemPrice * 0.1, 100)
				
	constructor(record: Record){
		this.data = record
	}
	
}
```
**🔻&nbsp;&nbsp;이름이 가격을 계산하는 price() 메서드의 범위를 넘어 클래스 전체 적용<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> 변수추출x, 함수추출o**

```kotlin
class Order{
	val data
	val quantity
		get() = this.data.quantity
	val itemPrice
		get() = this.data.itemPrice
	val basePrice
		get() = this.quantity * this.itemPrice
	val quantityDiscount
		get() = Math.max(0, this.quantity - 500) * this.itemPrice * 0.05
	val shipping
		get() = Math.min(this.quantity * this.itemPrice * 0.1, 100)	
	val price
		get() = this.basePrice - this.quantityDiscount + this.shipping
				
	constructor(record: Record){
		this.data = record
	}
	
}
```
<br>

### 📍 &nbsp;&nbsp;절차
① 추출하려는 표현식에 부작용은 없는지 확인<br>
② 불변 변수를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입<br>
③ 원본 표현식을 새로 만든 변수로 교체<br>
④ 테스트한다. <br>
⑤ 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체. 교체할 때마다 테스트


<br>
<div id='id-section4'/>

## 6.4 변수 인라인하기 Inline Variable

```kotlin
val basePrice = anOrder.basePrice
return (basePrice > 1000)
```
**🔻 변수 인라인**
```kotlin
return anOrder.basePrice > 1000
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM2ODg3MzI0OSwtNzczNDE3NDQyLDE5Mj
g4MTEzODAsMTcxMzcxOTQ1OSwtMTExNjI2ODU2OCwyMjUxOTEy
NjUsMTAxOTg3ODQ4NCwtMjQwMDQ3MDIsMTY3ODEyNTgyMCwtMT
U1NTg4ODMxMSwtNDEzNTQ2NjY4LDE2NDQ3NjI5OTgsNDU1OTg3
ODQxLDIwNjIzMzgyMzcsMTg1MzU2MTEyMCwxMjI2NjEyMjY2LC
0xMTAwOTAyNzkxLC0xNjYwNTk2ODEwLC0xNDg1NTM4MjA1XX0=

-->
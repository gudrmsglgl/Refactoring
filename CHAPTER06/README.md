# 기본적인 리팩터링
[함수 추출하기](#id-section1)<br>
[함수 인라인하기](#id-section2)<br>
[변수 추출하기](#id-section3)<br>
[변수 인라인하기](#id-section4)<br>
[함수 선언 바꾸기](#id-section5)<br>
[변수 캡슐화하기](#id-section6)<br>
[변수 이름 바꾸기](#id-section7)<br>
[매개변수 객체 만들기](#id-section8)<br>

### 저수준 리팩터링
- **추출**은 결국 이름 짓기이며, 코드 이해도가 높아지다 보면 이름을 바꿔야 할 때가 많다.
- **함수 선언 바꾸기** 
    - 함수의 이름을 변경할 때 많이 쓰인다.
    - 함수의 인수를 추가 제거할 때 적용
- **함수 이름 바꾸기**
    - 바꿀 대상이 변수일 때 
    - 변수 캡슐화하기와 관련 깊다.
- **매개변수 객체 만들기**
    - 자주 함께 뭉쳐 다니는 인수 
	    - [x] **객체 하나로 묶기**

### 고수준 리팩터링
- 함수를 만들고 나면 
	- [x] 다시 **고수준 모듈로 묶어야** 한다.
- 함수를 그룹으로 묶을 때
	- [x] **여러 함수를 클래스로 묶기**를 이용
- 읽기 전용 데이터
	- [x] **여러 함수를 변환 함수**로 묶기 
- 모듈들의 작업 처리 과정을 명확한 단계로 구분 
    - [x] **단계 쪼개기**

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
 #### ⭐&nbsp;&nbsp;**목적과 구현을 분리**
- 코드를 보고 무슨 일을 하는지 파악하는 데 한참이 걸린다면 
	- [x] 함수로 추출
	- [x] '무슨 일'에 걸맞는 이름을 짓는다.
		- [x] 😊 함수를 아주 짧게, 단 몇 줄만 담도록 작성하는 습관 
		- [x] 💭 함수 호출이 많아져서 성능이 느려질까 걱정하지 말아라
		- [x] 😫 5~6줄 이상 냄새 풍김
#### 길이를 기준
   - 한 화면을 넘어가면 안 된다
#### 재사용성 기준
   - 두 번 이상 사용될 코드는 함수로 만들고, 한 번만 쓰이는 코드는 인라인 상태로 놔두는 것.
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
		- ~~변수가 아닌 (주로)~~ **💫 함수로 추출**해야 한다.
		- 이름이 통용되는 문맥을 넓히면 다른 코드에서 사용할 수 있기 때문에 같은 표현식을 중복해서 작성하지 않아도 됨
		- 중복이 적으면서 의도가 잘 드러나는 코드를 작성 
<br>

### **ex) 변수 추출 시 함수를 벗어난 넓은 문맥에서 함수로 추출**<br>

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
<br>

### 🔎 &nbsp;&nbsp;변수를 인라인 할 때 
- 원래 표현식과 다를 바 없을 때 
- 변수가 주변 코드를 리팩터링하는 데 방해될 때 

<br>

### 📍 &nbsp;&nbsp;절차
① 대입문의 우변(표현식)에서 부작용이 생기지 않는지 확인<br>
② 변수가 불변으로 선언되지 않았다면 불변으로 만든 후 테스트한다.<br>
```
-> 이렇게 하면 변수에 값이 단 한번만 대입되는지 확인할 수 있다.
```
③ 이 변수를 가장 처음 사용하는 코드를 찾아서 대입문 우변의 코드로 바꾼다.<br>
④ 테스트한다. <br>
⑤ 변수를 사용하는 부분을 모두 교체할 때까지 이 과정을 반복한다.<br>
⑥ 변수 선언문과 대입문을 지운다.<br>
⑦ 테스트한다.

<br>
<div id='id-section5'/>

## 6.5 함수 선언 바꾸기 Change Function Declaration
```kotlin
fun circum(radius: Float){...}
```
**🔻 함수 선언 바꾸기**
```kotlin
fun circumference(radius: Float){...}
```
<br>

### 함수 선언
- 프로그램을 작은 부분으로 나누는 주된 수단
- 소프트웨어 시스템의 구성 요소를 조립하는 연결부 역할
- 잘 정의하면 새로운 부분을 추가가 쉬움, 잘못 정의하면 지속적인 방해 요인, 동작을 파악하기 어려워지고 요구사항이 바뀔 때 적절히 수정하기 어렵게 한다.

<br>

### 💫 &nbsp;&nbsp;중요한 것
- 함수 이름
```
* 이름이 좋으면 함수의 구현 코드를 살펴볼 필요 없이 호출물만 보고 무슨 일을 하는지 파악 가능
* 잘못된 함수를 발견하면 더 나은 이름이 떠오르는 즉시 바꾸자.
* 나중에 그 코드를 다시 볼 때 무슨 일을 하는지 '또' 고민하지 않게 됨.
```
> ##### 💡 좋은 이름을 떠올리는데 효과적인 방법<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;주석을 이용해 함수의 목적을 설명해보는 것. &nbsp;그러다 보면 주석이 멋진 이름으로 바뀌어 되돌아올 때가 있다.

<br>

- 매개변수 
```
* 함수가 외부 세계와 어우러지는 방식을 정의
* 다른 모듈과의 결합(coupling)을 제거
* 매개변수 올바르게 선택하는 것은 정답이 없고 어떻게 연결하는 것이 더 나은지 더 잘 이해하게 될 때마다 그에 맞게 코드를 개선할 수 있도록 함수 선언 바꾸기 친숙해져야 한다.
```

<br>

### 📍 &nbsp;&nbsp;간단한 절차
① 매개변수를 제거하려거든 먼저 **⚠️ 함수 본문에서 제거 대상 매개변수를 참조하는 곳**은 없는지 확인.<br>
② 메서드 선언을 원하는 형태로 바꾼다.<br>
③ 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정.<br>
④ 테스트한다. <br>


<br>

### 📍 &nbsp;&nbsp;마이그레이션 절차
① 이어지는 추출 단계를 수월하게 만들어야 한다면 함수의 본문을 적절히 리팩터링.<br>
② 함수 본문을 새로운 함수로 추출한다.<br>
```
-> 새로 만들 함수 이름이 기존 함수와 같다면 일단 검색하기 쉬운 이름을 임시로 붙여둔다.
```
③ 추출한 함수에 매개변수를 추가해야 한다면 '간단한 절차'를 따라 추가.<br>
④ 테스트한다. <br>
⑤ 기존 함수를 인라인한다. <br>
⑥ 이름을 임시로 붙여뒀다면 함수 선언 바꾸기를 한 번 더 적용해서 원래 이름으로 되돌린다.<br>
⑦ 테스트한다.

<br>

### **ex) 매개변수 추가하기**<br>
##### **요구사항: 예약 시 우선순위 큐를 지원하는 새로운 요구**

```kotlin
// Book 클래스..
fun addReservation(customer: Customer){
   this._reservations.push(customer)
}
```
② 함수 본문을 새로운 함수로 추출한다.
```kotlin
// Book 클래스..
fun addReservation(customer: Customer){
   this._reservations.push(customer)
}

fun zz_addReservation(customer: Customer){
   this._reservations.push(customer)
}
```
③ 새 함수의 선언문과 호출문에 원하는 매개변수를 추가한다.
```kotlin
// Book 클래스..
fun addReservation(customer: Customer){
   this.zz_addReservation(customer, false)
}

fun zz_addReservation(customer: Customer, isPriority: Boolean){
   assert(isPriority == true || isPriority == false) // 호출하는 곳에서 새로 추가한 매개변수를 실제로 사용하는지 확인
   this._reservations.push(customer)
}
```
⑤ 기존 함수를 인라인한다. <br>
⑥ 다 고쳤다면 새 함수의 이름을 기존 함수의 이름으로 바꾼다.

<br>

### **ex) 매개변수를 속성으로 바꾸기**<br>
##### **요구사항: 고객이 뉴잉글랜드에 살고 있는지 확인하는 함수**

```kotlin
fun inNewEngland(customer: Customer): Boolean{
   return ["MA","CT","ME","VT","NH","RI"].includes(customer.address.state)
}

val newEnglanders = someCustomers.filter{c -> inNewEngland(c)}
```
**🔻 함수 추출**

```kotlin
fun inNewEngland(customer: Customer): Boolean{
   val stateCode = customer.address.state
   return xxNewinNewEngland(stateCode)
}

fun xxNewinNewEngland(stateCode: StateCode): Boolean{
   return ["MA","CT","ME","VT","NH","RI"].includes(stateCode)
}
```
**🔻 함수 인라인 -> 기존 함수 호출문을 새 함수 호출문으로 교체**
```kotlin
val newEnglanders = someCustomers.filter{c -> xxNewinNewEngland(c.address.state)}
```
**🔻 함수 선언 바꾸기 -> 새 함수의 이름을 기존 함수의 이름으로 바꾼다.**
```kotlin
// 최상위
fun inNewEngland(stateCode: StateCode): Boolean{
   return ["MA","CT","ME","VT","NH","RI"].includes(stateCode)
}

// 호출문 
val newEnglanders = someCustomers.filter{c -> inNewEngland(c.address.state)}
```

<br>
<div id='id-section6'/>

## 6.6 변수 캡슐화하기 Encapsulate Variable

- 유효범위가 넓어질수록 다루기 어려워짐 -> 전역 데이터의 골칫거리 이유
- 접근할 수 있는 범위가 넓은 데이터를 옮길 때
	- 그 데이터로의 **💫 접근을 독점하는 함수를 만드는 식으로 캡슐화**하는 것이 가장 좋은 방법
	- 데이터 재구성이라는 어려운 작업을 함수 재구성이라는 더 단순한 작업으로 변환되는 것.
- 데이터를 변경하고 사용하는 코드를 감시하기 때문에 **데이터 변경 전 검증이나 변경 후 추가 로직을 쉽게** 끼워 넣을 수 있다. 
- 자주 사용하는 데이터에 대한 결합도가 높아지는 일을 막을 수 있다.
- private 유지(필드 캡슐화하기) -> 가시 범위 제한
- 불변 데이터는 가변 데이터보다 캡슐화할 이유가 적다. 데이터를 변경될 일이 없어서 갱신 전 검증 같은 추가 로직이 자리할 공간을 마련할 필요가 없기 때문.

<br>

### 📍 &nbsp;&nbsp;절차
① 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 만든다.<br>
② 정적 검사를 수행한다.<br>
③ 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 바꾼다. 하나씩 바꿀 때마다 테스트한다.<br>
④ 변수의 접근 범위를 제한한다. <br>
```
-> 변수로의 직접 접근을 막을 수 없을 때도 있다. 그럴 때는 변수 이름을 바꿔서 테스트해보면 해당 변수를 참조하는 곳을 쉽게 찾아낼 수 있다.
```
⑤ 테스트한다. <br>
⑥ 변수 값이 레코드라면 레코드 캡슐화하기를 적용할지 고려.<br>

<br>

### **ex) 전역 변수에 중요한 데이터가 담겨 있는 경우**<br>

```kotlin
var defaultOwner = "마틴 파울러"

// 다음과 같은 참조하는 코드..
spaceship.owner = defaultOwner

// 갱신하는 코드 역시 있을 것...
defaultOwner = "레베카 파손스"
```
**1. 기본적인 캡슐화를 위해 가장 먼저 데이터를 읽고 쓰는 함수 정의**
```kotlin
fun getDefaultOwner() = this.defaultOwner
fun setDefaultOwner(arg: String){this.defaultOwner = arg}
``` 
**3. 그런 다음 defaultOwner를 참조하는 코드를 찾아서 방금 만든 게터 함수를 호출하도록 고친다.**
```kotlin
spaceship.owner = getDefaultOwner()
setDefaultOwner("레베카 파슨스")
``` 

<div id='id-section6-ex1'/>

**4. 모든 참조를 수정했다면 변수의 가시 범위를 제한한다.** 
**🙆‍♀️ 1. Function set & get (캡슐화)**

```kotlin
private var defaultOwner = "마틴 파울러"  // 👈 가시 범위 제한
fun getDefaultOwner() = this.defaultOwner
fun setDefaultOwner(arg: String){this.defaultOwner = arg}
``` 
**🙆‍♀️ 2. Property set & get (캡슐화)**

```kotlin
private var _defaultOwner = "마틴 파울러"
var defaultOwner: String
   set(value){	
      this._defaultOwner = value
   }
   get() = this._defaultOwner
``` 
<br>

### 💊 &nbsp;&nbsp;값 캡슐화하기
- 기본 캡슐화 기법은 데이터 항목을 참조하는 부분만 캡슐화한다.
- 변수뿐 아니라 변수에 담긴 내용을 변경하는 행위까지 제어할 수 있게 캡슐화하고 싶을 때 
- 두 가지 방법
	- 값을 바꿀 수 없게 만드는 것.
	- 게터가 데이터의 복제본을 반환하도록 수정

<br>
<div id='id-section7'/>

## 6.7 변수 이름 바꾸기 Rename Variable
```kotlin
val a = height * width
```
**🔻 변수 이름 바꾸기**

```kotlin
val area = height * width
```

- 명확한 프로그래밍의 핵심 이름짓기.
- 변수는 프로그래머가 하려는 일에 관해 많은 것을 설명해준다. ( 단, 이름을 잘 지었을 때 )
- 한 줄짜리 람다식(laumbda expression)에서 사용하는 변수는 대체로 쉽게 파악 -> 한 글자로 된 이름
- 간단한 함수의 매개변수 이름도 짧게 지어도 될 때가 많다.
- 동적 타입 언어라면 이름 앞에 타입을 드러내는 스타일도 좋다. 
- 함수 호출 한 번으로 끝나지 않고 값이 영속되는 필드라면 이름에 더 신경 써야 한다.
<br>

### 📍 &nbsp;&nbsp;절차
① 폭넓게 쓰이는 변수라면 변수 캡슐화하기를 고려<br>
② 이름을 바꿀 변수를 참조하는 곳을 모두 찾아서, 하나씩 변경<br>
③ 테스트한다.

### **ex) 함수 밖에서 참조할 수 있는 변수 (읽기, 쓰기)**<br>
👉 &nbsp;&nbsp;[캡슐화로 처리](#id-section6-ex1)


<br>
<div id='id-section8'/>

## 6.8 매개변수 객체 만들기 Introduce Parameter Object
```kotlin
fun amountInvoiced(startDate: String, endDate: String){...}
fun amountReceived(startDate: String, endDate: String){...}
fun amountOverdue(startDate: String, endDate: String){...}
```
**🔻 매개변수 객체 만들기**

```kotlin
fun amountInvoiced(dateRange: DateRange){...}
fun amountReceived(dateRange: DateRange){...}
fun amountOverdue(dateRange: DateRange){...}
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMTQwNTg0ODMsMjAzNzA1ODc0Nyw1NT
E0NTgyNDIsMTQ1NzY0NDcyMiwtNDk0NzI1NzM2LDEzNzQ2Nzc4
MDIsLTYyODQ0NzAwOCwyNjk2MjQyOSwtNTA5ODI2ODAyLC0xNz
QxNzEyNTQsLTk3NDg5MjYxNSwtMjQ0Mzc4OTEyLDQ2NDQ5ODU4
LDIwNDYzMzE5ODAsLTE4NjkwODEyMTYsLTI5MDc4MDUyOCwtMT
AxMjkzOTQwNSwxMzM0OTYzOTAxLC00NzM4MTgxNTAsLTE4OTU4
ODA1ODddfQ==
-->
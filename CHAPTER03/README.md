# 코드에서 나는 악취
[기이한 이름](#id-section1)<br>
[중복 코드](#id-section2)<br>
[긴 함수](#id-section3)<br>

<div id='id-section1'/>

## 3.1 기이한 이름 Mysterious Name
> 코드를 명료하게 표현하는 데 가장 중요한 요소 하나는 '이름'
<br>함수, 모듈, 변수, 클래스 등은 그 이름만 보고 각각이 무슨 일을 하고 어떻게 사용해야 하는지 명확히 알 수 있도록 신경 써서 이름을 지어야 한다.

<br>
<div id='id-section2'/>

## 3.2 중복 코드 Duplicated Code

### 중복 코드의 예

    * 한 클래스에 딸린 두 메서드가 똑같은 표현식을 사용하는 경우
    -> 함수 추출하기 
    
    * 코드가 비슷하긴 한데 완전히 똑같지 않다면
    -> 문장 슬라이드하기로 비슷한 부분을 한곳에 모아 함수 추출하기를 더 쉽게 적용할 수 있는지 살펴본다. 

    * 같은 부모로부터 파생된 서브 클래스들에 코드가 중복
    -> 각자 따로 호출되지 않도록 메서드 올리기 적용해 부모로 옮긴다.


<br>
<div id='id-section3'/>

## 3.3 긴 함수 Long Function
> 코드를 이해하고, 공유하고, 선택하기 쉬워진다는 장점은 함수를 짧게 구성할 때 나오는 것.

### :black_nib: 짧은 함수 
##### :pencil2:&nbsp;좋은 이름

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyMDk2MTAxOCwtMTU3MDM4NjcwMV19
-->
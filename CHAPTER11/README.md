# API 리팩터링

- 소프트웨어 구성 빌딩 블록 - 모듈, 함수
	- 🧩 **블록들을 끼워 맞추는 연결부 - API**
		> 이해하기 쉽고 사용하기 쉽게 만드는 일은 중요한 동시에 어려움

- ### 좋은 API
	- 데이터를 **갱신 함수**,  **조회 함수**를 명확히 구분
		> - 섞여 있다면 👉🏻 질의 함수와 변경 함수 분리하기를 적용해 갈라야함.
		> - 값 하나로 여러개로 나뉜 함수들 👉🏻 함수 매개변수화하기 적용 하나로 합침.
		
	- 값 하나 때문에 여러 개로 나뉜 함수들	
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxNDEzNjg2NzcsMTYxMTQxNDk1MCwtMz
Y1MjU2MTA4XX0=
-->
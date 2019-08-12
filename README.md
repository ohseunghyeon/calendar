# 프로젝트 빌드, 실행 방법

## Prerequisite

1. node.js (v10.16.1 LTS) (https://nodejs.org/ko/)

2. yarn (https://yarnpkg.com/en/docs/install)

## Install dependencies

```
$ cd calendar-server & yarn

$ cd calendar-client & yarn
```

## Run

```
$ cd calendar-server & yarn dev

$ cd calendar-client & yarn start
```

# 문제 해결 전략

* 서버

    1. node.js 사용

    2. 메모리에 데이터를 저장하자

    3. 캘린더의 일정 데이터가 어떻게 생겼을지 생각하자
    
        * 제목, 시작시간, 끝 시간. 총 3 개의 항목 (추가로 primary key 용으로 id 하나)

    4. 일정 데이터는 4개의 컬럼으로 이루어져 있다. CRUD API가 필요하다

        * C: 4개의 컬럼을 가지고 만들면 된다.(id, 제목, 시작 시간, 끝 시간)
            * body: { 제목: String, 시작시간: unix timestamp, 끝시간: unix timestamp }
            * 시간을 체크하여 해당 시간에 다른 일정이 있는 경우 에러를 return

        * R: 읽기의 경우 시작, 끝 시간을 받아서 해당 범위의 일정들을 가져온다.
            * query param: ?시작시간&끝시간

        * U: 특정 일정을 선택하여 바꾸는 경우. 제목, 시간 다 바뀔 수 있다. 
            * body: { id: 1, 제목, 시작시간, 끝시간}
            * 시간을 체크하여 해당 시간에 다른 일정이 있는 경우 에러를 return

        * D: 일정 데이터를 ID로 식별하여 삭제한다.
            * body: { id: 1 }
    
* 클라이언트

    1. React 사용

    2. 화면을 어떻게 나눠서 컴포넌트를 구성할까?

        * 우선 상단의 control view와 하단의 calendar view로 나누고
        
        * 월/주 타입 별로 calendar view를 따로 둬야할 것 같다.
        
        * 일정을 추가, 수정, 삭제하는 용도로 사용될 팝업창은 월/주 타입과 관계 없이 하나고,

        * 이 팝업창에서 버튼은 어떻게?

            * 만들 때 취소, 저장 버튼
            * 수정할 때 취소 삭제 저장 버튼 (삭제 버튼만 조건에 따라 표시)

        * 요일은 일요일부터 시작하게 해서 한 주를 구성

        * Calendar view - 월
            
            * 위 아래로 나눠서
                * 위: 일 월 화 수 목 금 토
                * 아래: 각각 날짜와 일정을 나눠서 표시
            * 날짜가 1일인 경우 월을 함께 표시

        * Calendar view - 주

            * 위 아래로 나눠서
            * 위: 요일, 날짜 표시
            * 아래: 좌우로 나눠서
                * 좌: 시간을 매 시간별로 표시하게 하고
                * 우: 시간 별로 일정을 표시

    3. 데이터 관리를 어떻게 할까

        * global variable로 월(default)/주 의 값, 현재 보고 있는 년월주 값, 일정 데이터를 두자
        * 일정 데이터는 특정 flag를 두고 해당 flag가 변경됐고 true일 때 호출하자
        * 이 flag는 
            * 생성, 수정, 삭제 AJAX 호출 시 true로 변경하게 하자 GET 시에는 false로 다시 변경
            * 일정의 변경 시에도 true로 변경하게 하자
        * 일정 데이터 호출은 현재 보고 있는 year, month, week을 바탕으로 start, end unix timestamp를 생성하여 호출

    4. CSS는? 수작업
    
    5. 캘린더를 그리는 방법은?
        * 월 뷰: 현재 달의 첫 날을 가져온 뒤 그 날이 있는 주의 일요일을 가져온다. 다음으로 달의 마지막 날을 가져와서 그 주의 토요일을 가져오자. 이를 바탕으로 각 주를 div로 감싸서 그리고 각 날을 주 내부에 div로 그리면서 일 div에는 일정을 각각 넣어주면 될 것 같다.
        * 주 뷰: 월 뷰와 비슷한게 특정 날짜를 기준으로 일요일부터 토요일까지 가져오게 하자.

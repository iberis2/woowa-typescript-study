# woowa-typescript-study

## 스터디 소개
- 타입스크립트를 깊게 학습하기 위해 만든 스터디 입니다.
- ⟪우아한 타입스크립트⟫의 목차를 따라가면서 학습을 진행하되, 그 외의 다양한 참고 자료와 책들도 함께 공부하고 내용을 공유합니다.
- 일주일에 한 챕터를 읽고 관련된 내용에 대해서 모여서 토론합니다.

<br />

## 스터디 멤버
| 이름 | 참여 회수 |발표or업로드 : 🍎 참여 : 🍏|
|---|---|---|
| 김재정 | 4/7 |🍏🍏🍏🍎| 
| 박설화 | 6/7 |🍎🍎🍎🍏🍎🍎| 
| 심민섭 | 6/7 |🍏🍏🍎🍎🍎🍎|
| 양태현 | 6/7 |🍎🍎🍎🍎🍎🍎| 

<br />

## 스터디 일정

- 9/14 - 11/03 (총 7회)
- 주 1회 90분

| 회차 | 날짜 | 기록 | 참여자 |
|------|------|------|------|
| OT | ~~24-09-14(SAT) 20:00~~ | - | - |
| 1 | 24-09-21(SAT) 20:00 | [3장 고급 타입, 4장 타입 확장하기·좁히기](https://github.com/iberis2/woowa-typescript-study/tree/main/1%EC%A3%BC%EC%B0%A8) | **설화(발표)**, 민섭, 태현 |
| 2 | 24-09-28(SAT) 20:00 | [5장 타입 활용하기](https://github.com/iberis2/woowa-typescript-study/tree/main/2%EC%A3%BC%EC%B0%A8) |**태현(발표)**, 민섭, 설화, 재정 |
| 3 | 24-10-05(SAT) 20:00 | [6장 타입스크립트 컴파일, 7장 비동기 호출](https://github.com/iberis2/woowa-typescript-study/tree/main/3%EC%A3%BC%EC%B0%A8) | **민섭(발표)**, 설화, 재정, 태현 |
| 4 | 24-10-12(SAT) 20:00 | [8장 JSX에서 TSX로 ](https://github.com/iberis2/woowa-typescript-study/tree/main/4%EC%A3%BC%EC%B0%A8) | **태현(발표)**, 민섭, 설화, 재정 |
| 5 | 24-10-19(SAT) 20:00 | [9장 Hook](https://github.com/iberis2/woowa-typescript-study/tree/main/5%EC%A3%BC%EC%B0%A8) | **재정(발표)**, 민섭, 설화, 태현 |
| 6 | 24-10-26(SAT) 20:00 | [10장 상태관리, 11장 CSS-in-JS](https://github.com/iberis2/woowa-typescript-study/tree/main/6%EC%A3%BC%EC%B0%A8) | **민섭(발표)**, 설화, 태현 |
| 7 | 24-11-03(SAT) 20:00 | [12장 타입스크립트 프로젝트 관리, 13장 타입스크립트와 객체 지향](https://github.com/iberis2/woowa-typescript-study/tree/main/7%EC%A3%BC%EC%B0%A8) | (예정) **설화(발표)**, |

<br>

## 스터디 진행 방식

### 해당 주차의 '우아한 타입 스크립트' 학습
- 해당 주차의 정해진 분량을 학습
- 학습한 내용 정리 업로드
  - 자유로운 포맷(블로그 링크, ts파일, md파일 등)
  - 발표자 : 스터디 진행 1시간 전까지 PR (필수)
  - 참여자 : 스터디 진행한 당일 23:59 까지 PR (선택)

### 발표 및 토론
- 발표자가 해당 주차와 관련하여 준비한 내용을 발표
- 참여자는 학습하면서 궁금했던 부분을 서로 질문하거나 공유하고 싶은 내용 등을 자유롭게 공유
- 스터디 종료 전 당일 스터디 진행 관련 피드백

### 커밋 & 파일명 컨벤션
- 해당하는 주차의 폴더에 파일 작성
- 파일명 : {이름}_week_{해당주차}
ex) 박설화_week_01.md, 김재정_week_10.ts
- 커밋규칙 : {이름}: {해당주차}주차 학습 제출
ex) 박설화: 1주차 학습 제출

<br />

## 책 정보

![](https://contents.kyobobook.co.kr/sih/fit-in/400x0/pdt/9791169211567.jpg)\
[**우아한 타입스크립트 with 리액트, 우아한형제들 저자(글) 김민태 감수**](https://product.kyobobook.co.kr/detail/S000210716282)

<br />

### 목차
<details>
<summary>★ 이 책의 구성</summary> 
1장 들어가며
자바스크립트의 역사와 한계를 간단히 알아보면서 타입스크립트가 등장하게 된 배경을 살펴본다.

2장 타입
정적 타이핑을 하기 위해 타입스크립트가 제공하는 타입과 관련된 내용을 살펴본다. 타입이란 무엇이며 다른 언어에서 타입은 어떻게 동작하는지를 살펴보고, 타입스크립트의 타입을 어떻게 쓸 수 있는지 알아본다.

3장 고급 타입
자바스크립트 자료형에 없는 타입스크립트만의 타입 시스템을 소개한다. 그리고 타입의 개념을 응용하여 좀 더 심화한 타입 검사를 수행하는 데 필요한 지식을 살펴본다.

4장 타입 확장하기·좁히기
타입 확장과 타입 좁히기의 개념을 살펴보며 더욱 확장성 있고 명시적인 코드 작성법에 대해 알아본다.

5장 타입 활용하기
우아한형제들의 타입스크립트 활용 사례를 소개한다. 우아한형제들의 실무 코드 예시를 살펴보면서 정확한 타이핑을 하지 못해 발생하는 문제를 타입스크립트의 다양한 기법과 유틸리티 타입을 활용해 해결해본다.

6장 타입스크립트 컴파일
타입스크립트가 실행되는 전반적인 흐름을 살펴보고, 타입스크립트 컴파일러의 주요 역할과 구조에 대해 알아본다. 그리고 실제로 어떻게 컴파일하는지 확인해본다.

7장 비동기 호출
API를 요청하고 응답받는 행위는 모두 비동기로 이루어진다. 이 장에서는 타입스크립트에서 비동기 요청을 어떻게 처리하고 관리하는지를 다룬다.

8장 JSX에서 TSX로
리액트에서 사용하는 JSX 문법을 타입스크립트에 어떻게 적용하는지 소개한다.

9장 훅
리액트에서 제공하는 몇 가지 훅을 사용하여 상태 또는 사이드 이펙트를 다루는 방법을 소개한다. 또한 상태 로직을 재사용할 수 있게 해주고, 컴포넌트의 복잡성을 낮춰주는 커스텀 훅에 대해 알아본다.

10장 상태 관리
리액트 애플리케이션에서 가장 중요한 역할을 하는 상태에 대해 알아본다. 기본적인 상태의 개념을 익히고 어떻게 효율적으로 상태를 관리할 수 있는지를 살펴본다.

11장 CSS-in-JS
CSS-in-JS는 자바스크립트에서 CSS를 작성하는 방식이다. CSS-in-JS를 적용하면 CSS 스타일을 문서 레벨이 아니라 컴포넌트 레벨로 추상화해주기 때문에 관리가 용이해진다. 11장에서는 CSS-in-JS의 개념과 사용법에 관해 알아본다.

12장 타입스크립트 프로젝트 관리
타입스크립트 프로젝트에서 유용하게 활용할 수 있는 개념과 팁을 소개한다.

13장 타입스크립트와 객체 지향

</details>

<br />

### 참고 자료
- [우아한 타입스크립트 책에 사용된 예시 코드 저장소](https://github.com/woowa-typescript/woowahan-typescript-with-react-example-code)
- [타입스크립트 교과서, 조현영](https://product.kyobobook.co.kr/detail/S000208416779)

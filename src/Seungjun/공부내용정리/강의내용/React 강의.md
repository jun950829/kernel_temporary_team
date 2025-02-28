# React 특징

## 단방향 데이터 흐름
- 상위 컴포넌트에서 하위 컴포넌트로 데이터가 전달된다.

## 상태관리
- 상태가 변경되면 React는 컴포넌트를 리렌더링 한다.
- 기본적 상태관리 -> 기본 useState 훅
- 복잡한 상태관리 -> redux와 같은 상태 관리 라이브러리 사용

## 라이프사이클
- 생성 (mount) : 컴포넌트가 처음 DOM에 삽입될 때 
- 업데이트 (update): 컴포넌트의 상태가 props가 변경될 때
- 소멸 (unmount) : 컴포넌트가 DOM에서 제거될 때


## Virtual DOM
- DOM 자체 조작은 비용이 비싸다.
- 브라우저의 렌더링 엔진에 의해 수행
- 자바스크립트 객체이므로 읽고 쓰기가 빠르며 돔조작을 한 번에 모아서 반영

## 렌더링
- 새로운 Virtual DOM을 그리는 과정
- setState -> re-rendering 
- 자식 컴포넌트는 부모 컴포넌트가 렌더링되면 같이 리렌더링됌


## Reconciliation
- 기존 Virtual DOM 과 새로운 Virtual DOM을 비교하는 과정을 diffing이라고 함
- 기존 DOM과 비교하여 차이점만 갱신 시켜주는 전체과정을 뜻함 (Reconciliation)
- 성능 최적화의 핵심

### React는 휴리스틱 알고리즘으로 n시간 복잡도에 가까운 성능으로 비교함
- 타입: 다른 타입의 두 엘리먼트는 다른 트리를 만든다.
- Key: 각 랜더링에 유지되는 엘리먼트에 key를 통해서 같은 엘리먼트임을 알림


## Fiber
- react 16에서 추가된 개념
- 기존 stack reconciliation이 가진 문제를 해결하기 위한 새로운 아키텍처
- Main Thread에서는 기본적으로 렌더프로세스( HTML 파싱 -> 렌더 트리 생성 -> 레이아웃 -> 페인트 등등) , JS 실행, 사용자 상호작용 처리
- 자바 스크립트는 싱글 스레드이므로 하나의 코드 실행동안엔 다른 것이 실행되지 못함
- 일반적으로 display는 초당 60번 화면 갱신 1frame = 1000ms / 60 -> 16ms, 16ms 실행되는 js가 있다면 page junk가 발생
- 기존의 stack reconciliation작업은 동기적으로 실행되므로 콜 스택을 오래 차지하고 있어서 좋지 않았다.

## Fiber 요구사항
- 작업을 일시 중지하고 나중에 다시 돌아올 수 있다
- 작업의 우선 순위 지정
- 이전에 완료한 작업 재사용
- 더이상 작업이 불필요하면 제거

## 타입스크립트 장점
- 코드 품질 향상 및 버그 감소
- 개발 생산성 향상
- 구조화된 코드 작성
- 더 나은 코드 최적화 -> hidden 클래스 변경 최소화

## 컴포넌트를 잘 만들어야하는 이유
1. 재사용성
2. 가독성
3. 유지보수 용이
4. 테스트 용이
5. 성능 최적화

## 성능 최적화 기법
리액트의 렌더링이 시작되는 경우는
1. props나 state가 변경될 때
2. 부모가 렌더링 될 때

- 렌더링 최적화 -> 렌더링 종료지점 줄이기
- 자식 컴포넌트를 메모이제이션 ( const something = memo(컴포넌트)) -> children 컴포넌트 세팅과 같음

- useMemo: 큰 배열의 변경, 비싼 연산의 경우 사용하는 훅
- useCallback: 리렌더링 사이에 콜백 함수를 캐싱하는 훅


## useEffect

- 컴포넌트의 생명 주기 동안 side effect를 처리하는 훅

```
const MyComponent = () => {
    useEffect(() => {
        // side-effect code
        console.log('Component did mount or update');
        
        return (() => {
            console.log('Component will unmount or before next effect');
        };
    }, [ /* 의존성 배열 */]

    return <div>test<div>;
}

```

1. data fetching
2. dom 이벤트 리스너 추가 및 제거
3. 타이머 설정 및 해제

### 주의 사항
의존성 배열 관리
1. 필요한 모든 의존성을 포함해야한다.
2. 무한 루프 방지

### React Hook flow 
1. Mount ( Run lazy initializers )
useEffect나 useReducer와 같은 상태 초기화가 필요한 훅으로 인해 상태 초기화가 먼저 일어나고
초기화된 상태를 기반으로 렌더링 이후 커밋 단계에서 리액트는 돔에 해당 변경 사항들을 반영하게 되는데
현재 메인 스레드를 리액트가 사용하고 있어서 돔에는 반영되었지만 브라우저에 렌더링 되진않음
그 상태로 화면이 그려지기전에 useLayoutEffect에 작성된 내용이 실행되고 그다음 브라우저가 화면을 그림
마지막으로 useEffect에 작성된 내용 실행


2. Update
상태 초기화 로직 실행되지 않음. 변경된 상태를 기반으로 렌더링
render -> React updates DOM -> Cleanup LayoutEffects -> Run LayoutEffects -> Browser paints screen -> Cleanup Effects -> Run Effects

3. Unmount
useLayoutEffect의 hook 정리 함수 실행 useEffect의 정리 함수 실행


### useLayoutEffect가 useEffect보다 먼저 실행되는 이유
-> useLayoutEffect는 DOM 변경 사항이 화면에 그려지기 전에 동기적으로 실행되고 useEffect는 비동기적으로 실행되기전에 스케쥴러에 요청되므로 DOM이 화면에 그려진다음에 브라우저의 메인 스레드를 차지

### 부모 컴포넌트의 렌더링, useEffect와 자식컴포넌트의 랜더링, useEffect는 어떤 순서로 진행되는가
-> root부터 첫 번째 child를 탐색하면서 렌더링이 시작되고 더 이상 child를 찾지 못하면 sibling node로 건너가서 다음 child를 탐색하는 방향으로 진행. 렌더링 단계가 끝나면 자식부터 root까지 올라가면서 변경사항 등을 반영하고 effect를 실행하는 작업이 진행

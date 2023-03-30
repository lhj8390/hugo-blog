+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "Redux Toolkit(RTK) 개념 및 예제"
author = "lhj8390"
description = "Redux Toolkit 예제와 기존 Redux와 Redux Toolkit(RTK)의 차이점에 대해 설명한다."
date = "2022-11-09"
tags = ["frontend", "redux", "react.js", "typescript"]
categories = ["web"]
subcategories = ["react"]
+++
> Redux는 상태 관리 라이브러리로 전역 상태를 효과적으로 관리하는데 도움을 준다. <br/>
하지만 Redux를 제대로 활용하기 위해서는 많은 라이브러리를 추가적으로 설치해야 하는 불편함이 존재한다. 또한 많은 보일러플레이트 코드를 요구한다. <br/>
Redux 개발팀은 이 문제를 해결하기 위해 <span class="red">**Redux Toolkit(RTK)**</span> 를 만들었고, 특히 TypeScript를 통해 Redux를 개발할 경우 공식적으로 Redux Toolkit을 사용할 것을 추천하고 있다.
> 

## Redux Toolkit(RTK)이란?

---

Redux 개발 시 표준 로직이 되기 위해 고안된 패키지로, Redux 앱을 구축하는 데 필수적인 패키지와 기능이 포함되어 있다. action 생성, reducer 생성, store 설정, thunk 등을 생성할 때 코드를 쉽게 작성할 수 있는 함수를 제공한다.

**포함되는 기본 함수들**
- configureStore() : createStore를 랩핑하여 store 설정을 단순화한다.
- createReducer() : reducer를 생성하는 함수로, immer 라이브러리를 자동으로 사용하여 더 간단한 상태 변경이 가능하다.
- createAction() : action 타입과 action 생성자를 한번에 생성하도록 도와준다.
- createSlice() : reducer와 함께 그에 맞는 action 생성자와 action 타입을 자동으로 생성해준다.
- createAsyncThunk() : 비동기 action을 생성할 때 사용한다.


<aside>
Redux Toolkit은 Redux만 다루기 때문에 useSelector 나 useDispatch와 같은 React 구성 요소가 Redux store와 통신하기 위해서는 React-Redux 패키지가 필요하다.<br/>
</aside>

## Store 설정

---

기존 Redux에서 Store를 설정할 때는 다음과 같이 사용한다.

```tsx
import { combineReducers } from 'redux'
import testReducer from './reducers/testReducer'

// reducer를 결합하는 rootReducer 생성
const rootReducer = combineReducers({
   test: testReducer
});

export default rootReducer;
```

```tsx
import { createStore, applyMiddleware } from 'redux'
import { composeWithDevTools } from 'redux-devtools-extension'
import thunk from 'redux-thunk';
import rootReducer from './reducers/rootReducer'

const logger = createLogger({
    collapsed: true,
});
const middlewares = [thunk, logger];
const composedEnhancer = composeWithDevTools(applyMiddleware(...middlewares));

// rootReducer와 enhancer를 결합하여 store 생성
const store = createStore(rootReducer, composedEnhancer);
export default store;
```

1. combineReducers를 통해 reducers들을 하나로 묶는 rootReducer를 만든다.
2. middleware와 devtools를 하나로 합친 enhancer를 만든다.
3. createStore를 이용하여 store를 생성한다.

store를 설정할 때 이와 같이 필요한 패키지들이 많은 것을 알 수 있다. 하지만 Redux Toolkit을 사용하면 간편하게 strore를 설정할 수 있다.

### configureStore 사용

Redux Toolkit의 **configureStore**는 store 설정을 단순화하도록 도와준다.

```tsx
import { configureStore } from '@reduxjs/toolkit'
import { createLogger } from 'redux-logger';
import testReducer from './reducers/testReducer'

const loggers = createLogger({
    collapsed: true,
});

export const store = configureStore({
    reducer: {
      test: testReducer,
    },
    middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(loggers),
    devTools: true,
});
```

configureStore를 사용하면 다음과 같은 이점이 있다.

- reducer들을 rootReducer로 자동으로 변환해준다.
- thunk 미들웨어와 같은 기본적인 미들웨어를 자동적으로 추가한다.
- redux DevTools를 자동으로 연결해준다.

## action, reducer 생성

---

기존 Redux에서 action과 reducer를 생성할 때는 디렉터리를 따로 분리하여 action과 reducer 파일을 별도로 생성하였다.

```tsx
export enum TestActionTypes {
    TEST_SUCCESS = 'TEST_SUCCESS',
    TEST_FAILED = 'TEST_FAILED',
}

// 액션 선언
export const testActions = {
    testSuccess: () => ({ type: TestActionTypes.TEST_SUCCESS}),
    testFailed: (error: any) => ({ type: TestActionTypes.TEST_FAILED, payload: error}),
};

// 액션 타입 지정
export type TestAction = ReturnType<typeof testActions.testSuccess> | ReturnType<typeof testActions.testFailed>;
```

```tsx
type TestState = {
  initialized: boolean;
  test: string | null;
};

// 초기 상태 지정
const initialState: TestState = {
  initialized: false,
  test: null,
};

// 리듀서 생성
const testReducer = (state: TestState = initialState, action: TestAction) => {
  switch (action.type) {
    case TestActionTypes.TEST_SUCCESS:
      return { initialized: true };
    case TestActionTypes.TEST_FAILED:
      return { initialized: false, test: action.payload };
    default:
      return state;
  }
}

export default testReducer;
```

어플리케이션의 크기가 커진다면 action, reducer 코드 자체의 볼륨도 커져 관리가 힘들어진다. 이를 해결하기 위해 Redux Toolkit이 대안으로 내놓은 함수가 바로 `createSlice` 이다.

### createSlice 사용

createSlice는 세가지 옵션 필드가 존재한다.

- name : action이 생성될 때 접두사로 사용되는 문자열
- initialState : reducer의 초기 상태값
- reducers : 특정 액션이 수행될 경우에 대한 reducer 설정

createSlice를 사용한 예시이다.

```tsx
// 초기 상태값에 대한 인터페이스 선언
export interface TestState {
    initialized: boolean;
    test: string | null;
}

// 초기 상태값 설정
const defaultState: TestState = {
    initialized: false,
    test: null,
};

// crateSlice를 활용하여 action, reducer 생성
const testSlice = createSlice({
    name: 'TEST',
    initialState: defaultState,
    reducers: {
        [TestActionTypes.TEST_SUCCESS]: (state: TestState) => {
            state.initialized = true;
        },
        [TestActionTypes.TEST_FAILED]: (state: TestState, action: PayloadAction<string>) => {
            state.value = action.payload;
        },
    },
});

export default testSlice.reducer;
export const testSuccess = testSlice.actions[TestActionTypes.TEST_SUCCESS];
export const testFailed = testSlice.actions[TestActionTypes.TEST_FAILED];
```

reducer를 작성할 때 사용하던 switch 문 대신 객체 내부의 함수로 reducer를 작성할 수 있다. 또한 작성한 리듀서에 따라 액션이 자동으로 생성되어 별도로 액션을 생성할 필요가 없다.

## Thunks 작성하기

---

기존 비동기 API를 처리하기 위해서는 Redux-thunk 미들웨어를 사용하여 다음과 같은 코드로 구현했었다.

```tsx
export enum TestActionTypes {
    TEST_SUCCESS = 'TEST_SUCCESS',
}

// 액션 선언
export const testActions = {
    testSuccess: () => ({ type: TestActionTypes.TEST_SUCCESS}),
};

// 액션 타입 지정
export type TestAction = ReturnType<typeof testActions.testSuccess>;

export const test = (): ThunkAction< Promise<void>, RootState, {}, TestAction > => {
  // 3초 뒤 실행
  return async (dispatch: ThunkDispatch<{}, {}, TestAction>): Promise<void> => {
    return new Promise<void>((resolve) => {
        setTimeout(() => {
           dispatch(testActions.testSuccess());
        }, 3000);
    });
  };
};
```

ThunkAction의 타입을 계속 신경써줘야 하기 때문에 상당히 번거롭다. <span class="gray">(ThunkAction의 Custom 함수를 만들어 쓰는 방법도 있다.)</span> 이럴 경우, `createAsyncThunk` 를 사용하면 비동기 thunk 액션을 손쉽게 구현할 수 있다.

### createAsyncThunk 사용

createAsyncThunk를 두 가지 옵션 필드가 존재한다.

- action이 생성될 때 접두사로 사용되는 문자열
- Promise를 반환하는 콜백 함수

createAsyncThunk 를 사용한 예시이다.

```tsx
const testAsync = createAsyncThunk(
    TestActionTypes.TEST_SUCCESS,    // TEST_SUCCESS
    async () => {
        await new Promise((resolve) => setTimeout(resolve, 1000));
        return 'data';
    },
);
```

createAsyncThunk는 3개의 action creator와 action type를 생성하게 된다. 생성되는 action creator와 action type는 다음과 같다.

- testAsync.pending : TEST_SUCCESS/pending
- testAsync.fulfilled : TEST_SUCCESS/fulfilled
- testAsync.rejected : TEST_SUCCESS/rejected

단, 이렇게 만든 action은 createSlice 외부에서 구현한 것이기 때문에 createSlice의 **extraReducers** 옵션을 활용해야 한다.

```tsx
const testSlice = createSlice({
    name: 'TEST',
    initialState: defaultState,
    reducers: {},
    extraReducers: (builder) => {
        builder
          .addCase(testAsync.pending, (state) => {
            state.statue = "loading";
          })
          .addCase(testAsync.rejected, (state, action: PayloadAction<string>) => {
            state.value = action.payload;
          })
          .addCase(testAsync.fulfilled, (state, action: PayloadAction<string>) => {
            state.value = action.payload;
          });
    },
});
```

이제 createAsyncThunk로 생성한 액션으로부터 상태 값을 받아서 처리할 수 있다.

## Typed hooks 정의

---

Typed hooks를 정의해야 하는 이유는 다음과 같다.

1. useSelector를 사용할 경우 state 타입을 계속 지정해줘야 하는 번거로움이 존재한다. 
2. useDispatch를 사용할 경우 기본 Dispatch는 thunk 타입을 인식하지 못하기 때문에 thunk를 dispatch하기 위해서는 thunk 미들웨어가 추가된 store에서 dispatch를 직접 받아와야 한다.

configureStore로 만들어진 store를 이용해 RootState와 AppDispatch를 선언한다.

```tsx
export const store = configureStore({
    reducer: rootReducer,
    middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(loggers),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

store에서 정의된 RootState와 AppDispatch를 사용하여 Typed hooks를 정의한다.

```tsx
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux'
import type { RootState, AppDispatch } from './store'

export const useAppDispatch: () => AppDispatch = useDispatch
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector
```

이제 useSelector와 useDispatch 대신 useAppDispatch와 useAppSelector를 사용하면 된다.
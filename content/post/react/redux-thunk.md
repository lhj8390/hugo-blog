+++
author = "lhj8390"
title = "Redux-Thunk를 이용한 상태관리"
date = "2022-06-11"
description = "Redux-Thunk 사용 이유 및 작동 원리, 기본적인 적용방법에 대해 작성했다."
tags = ["frontend", "redux", "middleware", "react.js"]
categories = ["web"]
subcategories = ["react"]
+++

## 개요

---

> **Redux-Thunk**란 Redux에서 비동기 작업을 처리할 때 사용하는 미들웨어. \
> 액션 객체가 아닌 함수 자체를 dispatch할 수 있다는 특징이 있다.
> 
> Redux 미들웨어를 사용하면 액션이 dispatch 된 다음, 리듀서에서 해당 액션을 받아와 업데이트하기 전에 추가적인 작업을 할 수 있다.
> 
<br/>
<aside>
<strong>middleware란?</strong>

  dispatch 함수를 결합해서 새 dispatch 함수를 반환하는 고차함수.
  비동기 API 호출을 일련의 동기 action으로 바꾸는데 유용하다.

  즉, 미들웨어는 redux 저장소의 action을 변형하는 기능으로, 미들웨어를 통해 redux에 새로운 기능을 추가하며, 네트워크 요청을 만드는데 도움이 된다.
</aside>
<br/>

## Redux-Thunk 사용 이유

---

redux-thunk는 API 호출이 필요할 때 사용한다.

### async / await 호출

```jsx
export const fetchPost = () => {
  return async function(dispatch) {
    await axios.get('/api/post')
        .then((response) => {
            dispatch(getPostListSuccess(response));
        })
        .catch((error) => {
            dispatch(getPostListFailed(error));
        });
  }
}
```

Redux-thunk를 사용하지 않고 해당 코드로 action 호출 시 하단의 에러 코드가 노출된다.

<mark>action must be plain objects. Use custom middleware for async actions</mark> \
→ 액션 생성함수는 오직 plain object 형태만 리턴해야 하는데, 위와 같이 비동기적인 코드에선 함수도 리턴하기 때문.

<br/>
<aside>
<strong>console에 찍어보면 객체인데??</strong>

  console에 찍히는 response 객체는 해당 함수가 종료되고 return되는 값으로 
  리듀서로 dispatch되는 최초의 액션 형태는 plain이 아닌 async await 함수의 형태가 된다.
</aside>
<br/>

### promise로 호출


```jsx
export const fetchPost = async () => {
  const res = await axios.get('/api/post');
  
	return {
		type: 'GET_POST_LIST_SUCCESS',
		payload: res.data,
	};
};
```

액션 함수를 비동기적으로 처리를 하지 않는다면 에러는 발생하지 않지만, 데이터가 API Fetch되기 전에 액션이 리듀서에게 전달하기 때문에 data를 받아서 랜더링할 수 없다.

**Promise를 사용하면 순식간에 실행된 reducer에서 '반환된 데이터가 아직 없다'(객체에 아무것도 없다!)라고 판단한다.**

<br/>

## Redux-thunk 작동 원리

---

> 액션을 반환할 때 객체가 아니라 함수를 반환할 수 있게 해준다.
> 
> 
> 다른 말로 **액션이 함수의 형태일 때 그 함수를 실행하고 또 실행하여 객체가 되었을 때 reducer로 dispatch 한다.**
> 

**dispatch 이후에 Redux thunk가 실행된다.**

    ⚫ action creator -> action -> dispatch -> Redux Thunk


**Redux thunk에서 dispatch 된 액션이 함수인지 아닌지 판단한다.**

    ⚫ Redux Thunk -> 함수인가? -> 아니요.(Plain Object) -> dispatch -> Reducer

    ⚫ Redux Thunk -> 함수인가? -> 예 -> 함수실행 -> dispatch -> 함수인가? -> 아니요(Plain Object) -> Reducer


→ *즉, Redux-thunk가 API 요청이 끝날때까지 대기하고 반환값의 형태가 Plain Objects일 경우 이를 dispatch하는 역할을 담당한다.*

<br/>

## Redux 적용

---

### 미들웨어 설치

```bash
$ npm i redux-logger
$ npm i redux-thunk
$ npm i reudx-devtools-extension
```

스토어에 적용해주기 전에 라이브러리를 설치했다.

redux-thunk 뿐만 아니라 State, Props 등을 콘솔에서 보여주는 **Logger,** 크롬 익스텐션인 **Redux-devtools** 를 사용할 수 있도록 하는 **composeWithDevTools** 를 설치한다.


{{< figure src="/images/redux-thunk/1.png" alt="image" caption="Logger 적용 사진" class="medium" >}}

<br/>

### 미들웨어 적용

```jsx
import {applyMiddleware, compose, createStore} from 'redux';
import rootReducer from '../reducers';
import thunk from "redux-thunk";
import {createLogger} from "redux-logger/src";
import {composeWithDevTools} from "redux-devtools-extension";

const logger = createLogger();
const middlewares = [thunk, logger];  // thunk와 logger를 미들웨어에 추가

export default function () {
  const initialState = {};

  return createStore(
      rootReducer,
      initialState,
      compose(
          composeWithDevTools(applyMiddleware(...middlewares))  //middlewares 배열을 넣어주었다.
      )
  );
};
```

createStore 마지막 인자는 리덕스에 미들웨어를 추가할 수 있도록 도와준다.

middleware 배열에 thunk와 logger를 넣고 `applyMiddleware(...middlewares)` 를 사용하여 middleware 배열의 미들웨어를 사용하겠다고 명시한다.

크롬 익스텐션인 Redux-devtools를 사용하려면  applyMiddleware(..) 함수 자체를 composeWithDevTools로 감싸주면 된다.
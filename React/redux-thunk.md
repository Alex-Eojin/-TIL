# 리덕스 미들웨어

<br>

API 요청에 대한 상태를 잘관리 해야한다.

<br>

만약 요청이 시작했을때 로딩중,

요청이 성공했을때, 실패했을때 로딩이 끝났음을 명시 해주어야한다.

<br>

이러한 비동기 작업을 관리해야한다면 미들웨어를 사용하면 좋다.

<br>

리덕스, 리액트 리덕스, 리액트 액션 설치

```bash
npm i redux react-redux redux-actions
```

<br>

## 미들웨어란?

<br>

액션을 디스패치 했을때 리듀서에서 처리하기전에 미리 미들웨어를 처리한다.

<br>

미들웨어는 액션과 리듀서 사이의 중간 과정이라고 볼 수 있다.

<br>

액션 → 미들웨어 → 리듀서 → 스토어

<br>

미들웨어가 할 수 있는 작업은

1 . 전달 받은 액션을 단순히 **콘솔에 기록하거나**

2 . 전달 받은 액션 정보를 기반으로 **액션을 아예 취소 하거나**

3 . **다른 종류의 액션을 추가**로 디스패치에 전달 할 수 있다.

<br>

미들웨어는 이미 오픈소스로 많이 만들어져있다.

<br>

미들웨어 함수 구조

```jsx
const middleware = store = next = action => {}
```

<br>

함수안에 함수안에 함수를 반환한다.

<br>

store은 리덕스 스토어 인스턴스

next는 함수형태로 store.dispatch와 비슷한 역활

action은 액션

<br>

**주의!**

next가 store.dispatch와 비슷하다고 했다.

하지만 dispatch(action) 와 next(action)가 다른점이있다.

<br>

next(action)을하면 다음 처리해야할 미들웨어에게 액션을 넘겨준다.

만약없다면! 리듀서에게 액션을 넘겨준다.

```jsx
const loggerMiddleWare = (store) => (next) => (action) => {
  console.group(action && action.type);
  console.log('이전 상태', store.getState());
  console.log('액션', action);
  next(action); // 다음 미들웨어 혹은 리듀서에게 전달
  console.log('다음 상태', store.getState());
  console.groupEnd();
};

export default loggerMiddleWare;
```

<br>

미들웨어 적용하는 방법

```jsx
const store = createStore(rootReducer, applyMiddleware(loggerMiddleWare));
```

<br>

createStore의 두번째 매개변수에 applyMiddleware(미들웨어)함수를 넣어주어야한다.

index.js에서 store을 생성할때 같이 적용하면된다.

<br>

### redux-logger 사용

<br>

위에서 직접 logger 미들웨어를 만들었지만 이미 만들어진 라이브러리가 있다.

redux-looger를 사용하면 더 깔끔하고 잘만들어져 있다.

<br>

사용방법

```bash
npm i redux-logger
```

<br>

```jsx
const store = createStore(rootReducer, applyMiddleware(logger));
```

<br>

## 비동기 작업 처리하는 미들웨어 사용

<br>

**비동기 작업을 처리하는 미들웨어**

- redux-thunk: 비동기 작업을 처리할때 가장 많이 사용하는 미들웨어. 객체가 아니라 함수형태의 액션을 디스패치 할 수 있게 해준다.
- redux-saga: 특정 액션이 디스패치 되었을때 정해진 로직에 따라 다른 액션을 디스패치 시키는 규칙을 작성해 비동기 작업을 처리한다.

<br>

### redux-thunk

<br>

액션 생성 함수에서 일반 액션 객체를 반환하는 것이아닌 

함수를 반환하는것도 가능하게 해준다.

<br>

dispatch가 되었을때 바로 리듀서에서 작업되는 것이아닌

중간 미들웨어를 거치기 때문에 가능하다.

<br>

redux-thunk 코드를 보면 

```jsx
const thunk = ({ dispatch, getState }) => next => action => {
		typeof action === 'function' ? action(dispatch, getState) : next(action)
}
```

<br>

만약 action이 함수인경우 위와 같이 action 함수에 dispatch와 getState가 들어가 호출되어진다.

<br>

액션 함수에 첫번째 파라미터로 dispatch가 들어가고

두번째 파라미터로 getState가 들어간다.

<br>

덕분에 액션 함수는 dispatch, getState함수를 사용할 수 있게된다!

```jsx
const sampleThunk= () => (dispatch,getState) => {
	// 첫번째 파라미터덕분에 새 액션을 디스패치도 가능하다.
	// 두번째 파라미터덕분에 현재 상태 참조가능
}
```

<br>

설치 방법

```bash
npm i redux-thunk
```

<br>

적용방법

```bash
import ReduxThunk from 'redux-thunk';

const store = createStore(rootReducer, applyMiddleware(logger, ReduxThunk));
```

<br>

미들웨어에 적용해주면 된다.!

<br>

예제)

```jsx
export const increaseAsync = () => (dispatch) => {
  setTimeout(() => {
    dispatch(increase());
  }, 1000);
};

export const decreaseAsync = () => (dispatch) => {
  setTimeout(() => {
    dispatch(increase());
  }, 1000);
};
```

<br>

액션함수로 만들어서 dispatch에 액션함수를 넘길경우 redux-thunk 미들웨어에 의해 

dispatch를 파라미터 값으로 받아 올 수 있다.

<br>

따라서 위와 같이 setTimoute함수안에 dispatch를 사용할 수 있게된다.

<br>

### 웹 요청 비동기 작업 처리하기

<br>

예제) 리팩토리 전

modules/sample

```jsx
import { handleActions } from 'redux-actions';
import * as api from '../lib/api';

const GET_POST = 'sample/GET_POST';
const GET_POST_SUCCESS = 'simple/GET_POST_SUCCESS';
const GET_POST_FAILURE = 'simple/GET_POST_FAILURE';

const GET_USERS = 'sample/GET_USERS';
const GET_USERS_SUCCESS = 'sample/GET_USERS_SUCCESS';
const GET_USERS_FAILURE = 'sample/GET_USERS_FAILURE';

// thunk 함수 생성
// thunk 함수 시작, 성공, 실패 했을때 액션을 디스패치 한다.

export const getPost = (id) => async (dispatch) => {
  dispatch({ type: GET_POST }); // 요청 시작을 알림
  try {
    const response = await api.getPost(id);
    dispatch({
      type: GET_POST_SUCCESS,
      payload: response.data,
    }); // 요청 성공시때
  } catch (e) {
    // 요청 실패시
    dispatch({ type: GET_POST_FAILURE, payload: e, error: true });
    throw e;
  }
};

export const getUsers = () => async (dispatch) => {
  dispatch({ type: GET_USERS }); // 요청 시작을 알림
  try {
    const response = await api.getUsers();
    dispatch({
      type: GET_USERS_SUCCESS,
      payload: response.data,
    }); // 요청 성공시때
  } catch (e) {
    // 요청 실패시
    dispatch({ type: GET_USERS_FAILURE, payload: e, error: true });
    throw e;
  }
};

// 초기 상태
// 요청이 로딩중일떄는 loading이라는 객체에서 관리
const initalState = {
  loading: {
    GET_POST: false,
    GET_USERS: false,
  },
  post: null,
  users: null,
};

const sample = handleActions(
  {
    [GET_POST]: (state) => ({
      ...state,
      loading: {
        ...state.loading,
        GET_POST: true, // 요청 시작
      },
    }),
    [GET_POST_SUCCESS]: (state, action) => ({
      ...state,
      loading: {
        ...state.loading,
        GET_POST: false,
      },
      post: action.payload,
    }),
    [GET_POST_FAILURE]: (state) => ({
      ...state,
      loading: {
        ...state.loading,
        GET_POST: false,
      },
    }),
    [GET_USERS]: (state) => ({
      ...state,
      loading: {
        ...state.loading,
        GET_USERS: true, // 요청 시작
      },
    }),
    [GET_USERS_SUCCESS]: (state, action) => ({
      ...state,
      loading: {
        ...state.loading,
        GET_USERS: false,
      },
      users: action.payload,
    }),
    [GET_USERS_FAILURE]: (state) => ({
      ...state,
      loading: {
        ...state.loading,
        GET_USERS: false,
      },
    }),
  },
  initalState,
);

export default sample;
```

<br>

Samplecontainer

```jsx
import React, { useEffect } from 'react';
import { connect } from 'react-redux';
import Sample from '../components/Sample';
import { getPost, getUsers } from '../modules/sample';

const SampleContainer = ({
  post,
  users,
  loadingPost,
  loadingUsers,
  getPost,
  getUsers,
}) => {
  useEffect(() => {
    getPost(1);
    getUsers(1);
  }, [getUsers, getPost]);
  return (
    <Sample
      post={post}
      users={users}
      loadingPost={loadingPost}
      loadingUsers={loadingUsers}
    />
  );
};

export default connect(
  ({ sample }) => ({
    post: sample.post,
    users: sample.users,
    loadingPost: sample.loading.GET_POST,
    loadingUsers: sample.loading.GET_USERS,
  }),
  {
    getPost,
    getUsers,
  },
)(SampleContainer);
```

<br>

Samplecomponent

```jsx
import React from 'react';

const Sample = ({ loadingPost, post, loadingUsers, users }) => {
  return (
    <div>
      <section>
        <h1>포스트</h1>
        {loadingPost && '로딩 중...'}
        {!loadingPost && post && (
          <div>
            <h3>{post.title}</h3>
            <h3>{post.body}</h3>
          </div>
        )}
      </section>
      <hr />
      <section>
        <h1>사용자 목록</h1>
        {loadingUsers && '로딩 중...'}
        {!loadingUsers && users && (
          <ul>
            {users.map((user) => (
              <li key={user.id}>
                {user.username} ({user.email})
              </li>
            ))}
          </ul>
        )}
      </section>
    </div>
  );
};

export default Sample;
```

<br>

데이터를 불러와서 렌더링을 해줄때 유효성 검사를 해주는 것은 중요하다.

만약 위 코드에서 post객체가 유효할때만 그 내부의 title 과 body의 값을 보여준다.

<br>

users도 마찬가지로 데이터의 배열형태로 map함수로 JSX만들기전에 유효성검사를 시켜주어 오류를 발생시키지 않는다.

<br>

예제 리팩토링 후) 

<br>

lib/createRequest

```jsx
export default function createRequestThunk(type, request) {
  const SUCCESS = `${type}_SUCCESS`;
  const FAILURE = `${type}_FAILURE`;
  return (params) => async (dispatch) => {
    dispatch({ type });
    try {
      const response = await request(params);
      dispatch({
        type: SUCCESS,
        payload: response.data,
      });
    } catch (e) {
      dispatch({ type: FAILURE, payload: e, error: true });
      throw e;
    }
  };
}
```

lib에 따로 만든 유틸함수로 액션 타입과 API를 요청하는 함수만 넣으면 나머지 작업을 한줄로 하게 해준다.

<br>

이전 리팩토링 전의 코드에서 type부분만 변경해주면 되는 부분이므로

type을 파라미터 값으로 받아온다.

<br>

api.post와 api.users의 api불러오는 부분이 다르므로

그 부분만 request라는 파라미터 값으로 받아온다.

<br>

params라는 파라미터는 createRequestThunk가 호출되고 난후 반환되는 함수에 인수로 넣어주면된다.

<br>

이렇게 모듈에서 중복 코드는 lib에서 유틸함수로 만들어주면 유지보수에 좋다.

<br>

modules/loading

```jsx
import { createAction, handleActions } from 'redux-actions';

const START_LOADING = 'loading/START_LOADING';
const FINISH_LOADING = 'loading/FINISH_LOADING';

export const startLoading = createAction(
  START_LOADING,
  (requestType) => requestType,
);
export const finishLoading = createAction(
  FINISH_LOADING,
  (requestType) => requestType,
);

const initalState = {};

const loading = handleActions(
  {
    [START_LOADING]: (state, action) => ({
      ...state,
      [action.payload]: true,
    }),
    [FINISH_LOADING]: (state, action) => ({
      ...state,
      [action.payload]: false,
    }),
  },
  initalState,
);

export default loading;
```

loading 모듈만 따로 관리한다.

startLoading 일 경우 초기상태에서 action.playload를 키값으로 넣어 true로 직접 넣어주어 관리해준다.

<br>

initalState가 빈 객체이지만 rootReducer로 인해 합쳐지게 되므로

상태를 따로 컨테이너에서 불러와 관리할 수 있다. 

<br>

초기의 sample 모듈에서 초기상태값이 키의 값이 객체로 더 깊어지는 경우

상태를 따로 관리하는 편이 좋다.

<br>

modules/sample

```jsx
import { handleActions } from 'redux-actions';
import * as api from '../lib/api';
import createRequestThunk from '../lib/createRequestThunk';

const GET_POST = 'simple/GET_POST';
const GET_POST_SUCCESS = 'simple/GET_POST_SUCCESS';

const GET_USERS = 'sample/GET_USERS';
const GET_USERS_SUCCESS = 'sample/GET_USERS_SUCCESS';

// thunk 함수 생성
// thunk 함수 시작, 성공, 실패 했을때 액션을 디스패치 한다.

export const getPost = createRequestThunk(GET_POST, api.getPost);
export const getUsers = createRequestThunk(GET_USERS, api.getUsers);

// 초기 상태
// 요청이 로딩중일떄는 loading이라는 객체에서 관리
const initalState = {
  post: null,
  users: null,
};

const sample = handleActions(
  {
    [GET_POST_SUCCESS]: (state, action) => ({
      ...state,
      post: action.payload,
    }),

    [GET_USERS_SUCCESS]: (state, action) => ({
      ...state,
      users: action.payload,
    }),
  },
  initalState,
);

export default sample;
```

<br>

sample 모듈에서는 post와 users 의 성공한 경우에만 상태를 관리해 줄 수 있어 유지보수에 뛰어나다.

<br>

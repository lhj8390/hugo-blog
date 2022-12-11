+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "React - JWT를 활용한 API 인증(with Axios)"
author = "lhj8390"
description = "React axios를 사용해서 API 호출 시 JWT 인증을 수행하는 방법에 대해 기록한다."
date = "2022-12-11"
tags = ["frontend", "axios", "JWT", "react.js"]
categories = ["frontend"]
subcategories = ["React.js"]
+++

## 목표
---

> 권한이 있는 API에 액세스하려면 헤더에서 인증 관련 token을 함께 보내 권한이 있는 사용자인지 확인한다.
> 
> 만약 request에 토큰 값을 확인할 수 없을 경우 **401 (unauthorized)** 오류를 반환한다.<br/>
> Axios를 활용해 JWT 토큰으로 어떻게 인증을 수행하는지 기록해보았다.
> 

## 구현 전 점검사항

---

### 1. 로그인 완료 시 반환된 JWT 토큰이 저장되어 있는가?

로그인에 성공할 경우 해당 회원에 대한 JWT 토큰이 반환된다.<br/>
해당 JWT 토큰을 이용해 권한이 있는 API에 액세스할 수 있으므로 쿠키 또는 localStorage를 이용해 JWT를 저장해야 한다.

```jsx
await axios.post('/api/iam/login/', {
        email: data.email,
        password: data.password
    })
    .then((response) => {
        if(response.data.message === 'fail')
            dispatch(loginNotUser(response));
        else {
            // token setting
            const { token } = response.data;
            localStorage.setItem('id_token', token);
        }
    });
```

로그인을 성공할 경우 localStorage를 이용해 JWT 토큰을 저장했음을 알 수 있다.

### 2. API 부분에서 `authentication_classes`가 잘 설정되어 있는가?

 우선 View 단에서 호출될 인증 클래스가 잘 정의되어 있는지 확인이 필요하다.

`authentication_classes` 에는 

- **SessionAuthentication** : session framework를 통한 검증
- **TokenAuthentication** : Authorization token을 통한 검증
- **BasicAuthentication**: 사용자의 사용자 이름과 암호로 서명된 HTTP 기본 인증

주로 3가지 방식이 존재하는데, 우리는 JWT 토큰 인증 방식을 구현할 예정이기 때문에 과감히 지워준다.

```python
class PostViewSet(viewsets.ModelViewSet):
    # authentication_classes = [TokenAuthentication, SessionAuthentication]
    permission_classes = [IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly]
```

## Axios에서 인증 헤더 설정

---

> 이제 localStorage에 저장된 JWT 토큰값을 사용하여 권한이 있는 API 에 JWT를 인증 헤더와 함께 전송한다.
> 
> API는 Authorization 헤더를 읽고 인증된 사용자인지 판단한다.
> 

```jsx
await axios.post('/api/post/', data, {
        headers: { 'Authorization' : 'JWT '+ localStorage.getItem('id_token') }
    })
    .then((response) => {
        dispatch(createPostSuccess(response));
    })
    .catch((error) => {
        dispatch(createPostFailed(error));
    });
```

JWT 토큰을 통해 인증을 수행할 경우 

헤더 부분에 **Authorization** : “**JWT” 텍스트와 토큰 값을 함께 넣어주어야 한다.**

{{<figure src="/images/react-authorized-jwt-with-axios/1.png" class="large" caption="*이런식으로*">}}

토큰 앞부분에 JWT를 명시해주지 않을 경우 JWT 토큰으로 인식하지 못하여 401 error를 반환한다.<br/>

{{<figure src="/images/react-authorized-jwt-with-axios/2.png" class="left">}}

<br/><br/><br/>

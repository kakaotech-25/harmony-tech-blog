---
title: Next.js에서의 라우팅
date: "2024-10-27"
writer: 리안
tags:
  - React
  - Next
  - App Router
  - Page Router
  - 리안
previewImage: Next.png
---

모행 프로젝트를 진행하면서 라우팅에 대한 논의의 시간을 가졌고 현재 대부분의 프론트엔드 개발자들은 Next.js를 많이 사용하고 있어서 리액트에서 react-router-dom을 사용하지 않는다는 것을 알게 되었다.

그래서 Next.js에서는 어떤 라우팅 방식을 사용할 수 있는지 알아보고자 한다.

## 모행의 라우팅 방식

앞서 작성했던 글에서 확인할 수 있듯이 모행은 아래와 같이 라우팅을 하고 있다.

```js
import "./App.css"
import { Routes, Route } from "react-router-dom"
import Header from "./components/Header/Header"
import Footer from "./components/Footer/Footer"
import Home from "./pages/Home/Home.jsx"
import Login from "./pages/Login/Login.jsx"
import Profile from "./pages/Profile/Profile.jsx"
import LivingInfo from "./pages/LivingInfo/LivingInfo.jsx"
import InterestedPlace from "./pages/InterestedPlace/InterestedPlace.jsx"
import Mypage from "./pages/Mypage/Mypage.jsx"
import Planner from "./pages/Planner/Planner.jsx"
import PlannerDetails from "./pages/PlannerDetails/PlannerDetails.jsx"
import TravelDetails from "./pages/TravelDetails/TravelDetails.jsx"
import Landing from "./pages/Landing/Landing.jsx"
import Callback from "./pages/Login/Callback.jsx"

function App() {
  return (
    <>
      <Header />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/callback/kakao" element={<Callback />} />
        <Route path="/landing" element={<Landing />} />
        <Route path="/login" element={<Login />} />
        <Route path="/signup/profile" element={<Profile />} />
        <Route path="/signup/livinginfo" element={<LivingInfo />} />
        <Route path="/signup/interestedplace" element={<InterestedPlace />} />
        <Route path="/mypage" element={<Mypage />} />
        <Route path="/traveldetails/:id" element={<TravelDetails />} />
        <Route path="/planner" element={<Planner />} />
        <Route path="/plannerdetails/:id" element={<PlannerDetails />} />
      </Routes>
      <Footer />
    </>
  )
}

export default App
```

위와 같이 react-router-dom을 사용하면서 여러 페이지를 import해야 하고 폴더구조를 빠르게 파악하기 어려웠던 경험이 있다.

Next.js는 파일 기반으로 라우팅을 하기에 파일만 생성하여 자동으로 페이지를 등록할 수 있어 따로 import를 하지 않아도 되고 폴더 구조와 파일경로만으로 구조를 빠르게 파악하기 좋다는 장점이 있다.

폴더 기반 라우팅 방식을 가지고 있는 Next.js도 두가지의 라우팅 방식을 가지고 있다.

첫 번째는 페이지 라우팅으로 12버전 이전까지 사용되던 방식이고 두 번째는 앱 라우팅으로 13버전에서 새롭게 도입된 방식이다.

현재는 앱 라우팅 방식으로 많이 변화되고 있는 추세이지만 아직까지도 페이지 라우팅 방식을 사용하는 기업이 많기 때문에 두 가지 방식에 대한 기본적인 개념과 차이점에 대해 알아보고자 한다.

## Next.js의 Page Router

페이지 라우팅은 현재 많은 기업에서 사용되고 있는 안정적인 라우터로 React Router 처럼 페이지 라우팅 기능을 제공한다. 따라서 pages 폴더의 구조를 기반으로 페이지를 라우팅해준다.

예를 들어 Next App에 pages라는 폴더가 있고 해당 폴더 아래에 자바스크립트 파일을 저장해두면 자동으로 이 파일들의 경로와 이름에 따라서 페이지 라우팅이 제공된다.

![페이지라우팅 구조사진](../../page-router.png)

또한 동적 경로에 대응하는 페이지도 쉽게 만들 수 있는데 item이라는 경로 아래에 대괄호오 [id].js를 만들어주면 이 페이지는 동적경로에 대응하는 페이지의 역할을 하게 된다.
가변적인 값이 대괄호 속에 있는 아이디에 하나씩 매핑되는 식으로 라우팅 처리가 된다고 이해하면 된다.
![동적경로 라우팅](../../dynamic-routes.png)

이렇듯 Next.js의 파일기반 라우팅은 파일 이름과 폴더 구조만으로 라우트를 생성하여 경로가 자동으로 연결되기 때문에 react-router-dom 처럼 라우트마다 코드를 추가할 필요가 없어 유지보수가 간편하다.

추가적으로 Next.js에서는 404.js, \_app.js, \_document.js 파일을 사용할 수 있다.

> 404.js : 정의되어 있는 라우트 이외의 경로를 요청할 때 표시되는 404페이지를 의미한다.  
> \_app.js : 모든 페이지에 공통 레이아웃이나 스타일을 적용할 때 사용된다.  
> \_document.js : HTML과 <head> 태그 설정을 위한 커스텀 문서 컴포넌트다.

또한 폴더를 중첩하여 URL 구조를 쉽게 만들 수 있다.

> pages/blog/index.js -> /blog  
> pages/blog/post.js -> /blog/post

위와 같이 폴더를 중첩함으로써 복잡한 경로도 면확하게 구성할 수 있어 구조를 파악하기가 쉽다.  
이렇듯 Next.js의 12버전까지는 페이지 라우팅을 통해 react-router-dom과는 다른 구조의 라우팅을 제공했다.  
페이지 라우팅도 충분히 편리한 라우팅 방식을 제공하고 있지만 13버전 이후에 나온 App Router는 더욱 실용적인 라우팅 방식을 제공한다고 한다.

## Next.js의 App Router

앱 라우팅에서는 pages 폴더 아래에 있는 파일만 페이지로 취급했던 페이지 라우팅과는 다르게 무조건 page라는 이름을 갖는 파일만 경로로써 취급된다.  
그래서 search라는 경로에 대응하는 페이지를 만들기 위해서는 search라는 폴더아래 page.js와 같은 파일을 만들어야 한다.
![앱라우팅 구조사진](../../app-router.png)

동적 경로를 설정할 때에도 [id] 폴더를 만들고 그 안에 page.js라는 파일을 만들어줘야 동적 경로에 대응하는 페이지를 생성할 수 있다.
![동적경로 라우팅2](../../dynamic-routes2.png)

추가적으로 앱 라우팅에서는 기본적으로 서버 컴포넌트로 렌더링이 되며 클라이언트 렌더링이 필요한 경우 "use client" 를 통해 클라이언트 컴포넌트를 지정할 수 있다.

또한 앱 라우팅에서는 폴더마다 layout.js 파일을 생성하여 경로마다 레이아웃을 다르게 적용할 수 있다. 이를 통해 중첩된 레이아웃 구성이 가ㄴㅇ해지고 한 페이지의 레이아웃만 변경해도 다른 페이지의 레이아웃에는 영향을 미치지 않도록 할 수 있다.

### Page Router와 App Router 요약

|              | 페이지 라우팅(Page Router)                 | 앱 라우팅(App Router)                                         |
| ------------ | ------------------------------------------ | ------------------------------------------------------------- |
| 폴더 위치    | pages 폴더 아래                            | app 폴더 아래                                                 |
| Next.js 버전 | 12버전 이하에서 주로 사용                  | 13버전 이상에서 주로 사용                                     |
| 라우팅 방식  | 파일 기반 라우팅, 파일명이 URL 경로와 매핑 | 파일과 폴더 기반 라우팅 폴더와 파일 구조가 경로 설정          |
| 컴포넌트     | 클라이언트 컴포넌트만 사용                 | 서버 컴포넌트가 기본, "use client"로 클라이언트 컴포넌트 지정 |
| 코드 분할    | 페이지 단위로 코드 분할 가능               | 페이지, 컴포넌트 별 코드 분할 가능                            |

> 출처 : 한 입 크기로 잘라먹는 Next.js

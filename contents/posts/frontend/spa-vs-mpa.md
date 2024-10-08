---
title: SPA와 MPA 비교
date: "2024-09-29"
writer: 리안
tags:
  - React
  - SPA
  - MPA
  - 리안
previewImage: how.png
---

프로젝트를 시작하기 앞서 우리 서비스에는 SPA가 적합한지 MPA가 적합한지에 대한 조사를 진행했다.
조사하면서 알게된 정보를 정리해보려고 한다.

## SPA(Single Page Application)
SPA는 단일 페이지를 기반으로 하는 웹 애플리케이션이다. 사용자가 앱을 처음 로드할 때, 모든 필요한 부분을 모두 가져오고 이후에 새로운 페이지를 로드하지 않고 하나의 페이지 안에서 필요한 페이지를 동적으로 업데이트하는 방식으로 동작한다.

페이지 전환 시 서버에서 새로운 페이지를 불러오는 것이 아니라 JavaScript로 필요한 데이터를 요청해 부분적으로 갱신하는 동작방식을 가지고 있기에 페이지 전환이 빨라 사용자 경험이 매우 부드럽다는 장점이 있다. 또한, 한 번 로드한 리소스를 재사용하고 필요한 데이터만 서버에서 요청하므로 네크워크 사용이 효율적이다. 주로 클라이언트 측에서 렌더링을 담당하는 CSR 방식으로 초기 로딩 이후 성능이 빠르다.

하지만 서버에서 완전한 페이지를 제공하지 않기에 검색엔진최적화를 의미하는 SEO에 불리하고 애플리케이션 전체를 처음 로드할 때 모두 불러오기에 초기 로딩 시간이 길고 클라이언트 측에서 모든 상태를 관리해야 하기 때문에 애플리케이션이 복잡해질 수 있다는 단점도 있다.

## MPA(Multi Page Application)
MPA는 여러 개의 페이지로 만들어진 웹 애플리케이션이다. 각각의 페이지가 고유한 URL을 가지며 페이지 전환마다 서버에서 새로운 페이지를 요청해 렌더링된다.

페이지가 독립적으로 관리되기 때문에 상태관리나 라우팅이 간단하고 각 페이지에서 서버 요청이 이루어지므로 보안에 유리하며 서버 측에서 완전한 페이지를 제공하므로 SEO에 최적화되어 있다. 대부분의 렌더링이 서버 측에서 이루어지는 SSR 방식으로 브라우저는 완전한 페이지를 받아서 표시한다.

하지만 각 페이지를 서버에서 새로 불러오기에 페이지 전환 시 로딩 시간이 발생할 수 있고 서버 사이드 개발이 복잡해질 수 있으며 네트워크 효율이 떨어질 수 있다는 단점이 있다.

## SPA VS MPA 요약
|              | SPA                                   | MPA                                   |
|--------------|---------------------------------------|---------------------------------------|
| 개념          | 한 개의 페이지로 구성된 애플리케이션           | 여러 개의 페이지로 구성된 애플리케이션          |
| 로딩 속도      | 초기 느림, 이후 빠름                       | 초기 빠름, 이후 느림                       |
| 상태 관리      | 클라이언트에서 복잡하게 처리                  | 서버에서 페이지별로 처리                     |
| 렌더링         | CSR (Client-Side Rendering)           | SSR (Server-Side Rendering)           |
| SEO          | 기본적으로 불리함 (SSR로 보완 가능)           | SEO에 유리                              |
| 사용 예시      | Gmail, Facebook, Twitter              | Amazon, eBay, 네이버                     |


## 모행 서비스에는 어떤 게 적합할까?
우선 우리 프로젝트는 React로 구현할 예정이었다. React를 사용하는 경우에는 SPA가 더 적합하다. React는 기본적으로 클라이언트 측에서 컴포넌트 기반의 UI 렌더링을 제공하며 SPA 핵심요소인 빠른 페이지 전환과 동적인 사용자 경험을 강화하는데 최적화되어 있다. 모행 서비스는 많은 페이지를 가지고 있지 않기에 초기에 모든 페이지를 불러와 로딩속도가 느려질 수도 있다는 단점도 적용되지 않는 부분이라고 생각했다.

**[실제 모행 서비스의 SPA 라우팅 구현]**
```js
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
  );
}
```

## 추가적으로 고려해보아야 할 부분
현재는 SEO가 중요한 부분이 아니기에 SPA 방식도 괜찮다고 판단했지만 추후 SEO가 중요한 경우에는 React에서도 SSR을 적용하는 **Next.js** 같은 프레임워크를 사용하는 것이 좋다. 이 방식을 통해 초기 페이지 로딩 속도를 개선하고 SEO 문제를 보완할 수 있다. 또한 클라이언트 측에서 상태관리를 쉽게 할 수 있도록 **Redux, Context API**를 사용하는 것이 좋다.

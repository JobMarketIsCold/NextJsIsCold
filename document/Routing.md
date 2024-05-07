# 새로운 기능 배우기

저번 시간에 layout에 대해서 말해본다 하고 끝이 났었는데, layout은 나중에 필요해지면 설명을 할 것이다.
우리가 이제부터 배울 것들을 이해하고 나면 layout이 왜 필요한지 알 수 있게 될 것이다.

## 새로운 페이지 만들기 (new page)

React에서는 라우팅을 하기 위해서 react-router를 사용했었다.
react-router는 url을 지정하고 컴포넌트를 불러오는 형식이였다.

하지만, NextJs에서는 우리가 직접 url을 적어줄 필요가 없다.
대신 우리는 파일 시스템을 사용하여 url을 표현할 것이다.

현재 우리는 app/page.tsx밖에 없다. 이것을 root segment라고 한다.
왜냐하면, 이 페이지가 유저가 가장 먼저보게되는 ("/") 루트 페이지이기 때문이다.

그럼 우리가 about-us라는 페이지를 만들기 위해서는 어떤 행위를 해야 할까?
폴더를 about-us라는 이름으로 만들어주면 NextJs에게 이 폴더명 하나의 페이지가 될 수 있다고 알려주는 것이다. 그리고 그 폴더안에 page.tsx를 만들고 컴포넌트를 리턴해주면 제대로 작동하는 것을 볼 수 있다.

NextJs는 이렇게 간단하게 파일 시스템으로 경로를 표시할 수 있다!

만약에 우리가 a/b/c와 같은 깊은 url 구조를 만들었을 때, page.tsx가 없다면 경로의 일부분이 된다.

그럼 app 폴더에 다른 파일은 만들면 안될까?
아니다, page라는 파일만 만들지 않는다면 렌더링 되지 않기에 문제되지 않는다.

결국 사용자들이 보게되는 화면은 page.tsx이다.

## client, server-component, layout

app의 모든 페이지에서 사용하는 네비게이션 바를 구현하면서 server-component와 layout의 개념을 배우고 layout파일이 필요한 이유를 알아볼 것이다.

### not-found

일단 저번에 말하지 못했던 특별한 파일명(page, layout같은)인 not-found 파일이다.

우리가 주소창에 아무 주소가 치게되면 404오류를 보여주게 된다.
not-found.tsx를 app에 만들어주고 컴포넌트를 반환해주면 not-found페이지에서 우리가 반환한 컴포넌트를 렌더링해준다.

---

그럼 전역에서 사용하는 네비게이션 바를 구현해보자

프로젝트의 component폴더를 만들어주고 navigation.tsx를 만든다.

```TSX
export default function Navigation() {
  return (
    <nav>
      <ul>
        <li>Home</li>
        <li>About Us</li>
      </ul>
    </nav>
  );
}
```

위 코드를 작성해준다.

그럼 어떤식으로 url을 이동할 수 있을까?
a태그로도 가능하지만 이것은 next의 navigation을 사용하는 것이 아닌 브라우저의 navigation을 사용하는 것이기 때문에 우리가 원하는바가 아니다.

우리는 react-router에서 했던것과 비슷하게 Link를 사용할 것이다.

```TSX
import Link from "next/link";

export default function Navigation() {
  return (
    <nav>
      <ul>
        <li>
          <Link href="/">Home</Link>
        </li>
        <li>
          <Link href="/about-us">About Us</Link>
        </li>
      </ul>
    </nav>
  );
}
```

이런식으로 코드를 작성하고 일단 잘 작동이 되는지 확인하기위해서 복붙을 하여 확인해보자.

일단 client와 server component를 알아보기전에 우리가 home에 있으면 home옆에 이모지를 넣어 현재 어디있는지 알게하는 코드를 작성해보자.

그러기 위해 우리 경로명을 알려주는 hook을 사용할 것이다.
NextJs에는 url정보를 알려주는 hook이 존재한다.

우리가 사용할 hook은 usePathname이라는 hook이다.
사용을 해보면 오류가 뜰 것인데, 바로 client component에서만 사용할 수 있다는 것이다. 그래서 useClient라는 문구를 파일 상단에 적으라고 한다.

```TSX
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";

export default function Navigation() {
  const path = usePathname();
  return (
    <nav>
      <ul>
        <li>
          <Link href="/">Home {path === "/" ? "🤓" : ""}</Link>
        </li>
        <li>
          <Link href="/about-us">About Us</Link>
        </li>
      </ul>
    </nav>
  );
}
```

이런식으로 해결은 하였지만 우리가 어떻게 해결을 했는지를 알아야한다.

이제 client component와 server component에 대해서 알아볼 것인데, 그 전에 NextJs가 어떤 방식으로 애플리케이션을 렌더링하는 이해해야 한다. (React와 비교해서)

## CSR vs SSR

렌더링이란?  
NextJS가 리액트 컴포넌트를 브라우저가 이해할 수 있는 html로 변환하는 작업이다.

### CSR (ClientSideRendering)

리액트가 렌더링되는 과정은 CSR이다.
CSR에서는 모든 렌더링이 클라이언트 측에서 발생한다.

이 방식에는 단점이 존재한다.
**첫 번째 단점**
브라우저에서 react code를 보게 되면 처음에는 스크립트가 비어있는 것을 알 수 있는데, 왜냐하면 처음에는 빈 html만 보내고 나중에 JS를 로드해서 렌더링이 되는 것이기 때문이다.

그래서 JS를 다 로드하기 전까지의 공백이 생긴다. (빈 화면)
만약에 사이트에 JS를 꺼두었다먼 아무 화면도 볼 수 없게 된다. (이런 상황은 거의 없다.)
하지만, 데이터 연결 상태가 좋지 않은 상태에서 들어오게 된다면 유저는 JS를 모두 다운로드 할 때까지 빈화면을 보고 있어야하는 상황이 생기게 된다.

**두 번째 단점**
SEO 검색 엔진 최적화이다.
만약 우리 웹사이트가 구글 검색엔지에 노출되고 싶다면 google에게 빈 페이지를 보여주지 않는 것이 좋다. 구글은 우리 사이트의 html을 보기 때문이다.

이것이 바로 CSR이다.

> 클라이언트는 자바스크립트를 로드하고, 자바스크립트가 UI를 빌드한다.

### SSR (ServerSideRendering)

NextJs를 사용하게 되면 자동으로 SSR이 된다.

NextJs를 사용한 웹사이트의 소스코드를 보면 페이지의 내용들이 실제로 소스코드에 들어가있는 것을 볼 수 있다. 즉, 자바스크립트가 없어도 내용이 보인다는 것이다.

NextJs의 모든 page의 컴포넌트를 NextJs에서 최우선으로 server에서 렌더링 해준다.
NextJs는 사용자에게 html을 주기전에 server에서 application을 렌더링해주고 그 html을 브라우저의 request에 대한 response로 전달해준다.

이것은 엄청난 이점이 된다.

> 모든 컴포넌트와 페이지들은 server(backend)에서 먼저 render된다.
> ('use client' 사용 여부와 상관없음)

---

## Hydration

이제 CSR과 SSR을 알아보았으니 그 이후에는 어떤 일이 발생하는지를 알아볼 것이다.

**하이드레이션(Hydration)이란?**

> 서버사이드 렌더링(SSR)을 통해 만들어 진 인터랙티브 하지 않는 HTML을 클라이언트 측 자바스크립트를 사용하여 인터랙티브한 리액트 컴포넌트로 변환하는 과정을 말한다.
> (서버 환경에서 이미 렌더링된 HTML에 React를 붙이는 것)

자바스크립트가 비활성화된 상태에서 페이지 이동을 하면 a태그의 href만을 사용하기에 hard navigation을 하게된다.
그리고 만약 자바스크립트를 활성화하게 되면, 새로고침이 되지 않고 빨라진다.
이게 바로 react가 hydration된 것이다.

처음에는 a였다 react-component로 변환 된 것이다. 그래서 페이지 전체를 reload하지 않고 빠르게 navigate가 가능하다.
즉, client side rendering을 하는 것이다.

> \<Link> component를 사용하며 브라우저가 전체 페이지를 로드하지 않고  js 상에서 page component를 교체한다.

간단설명

> hydration은 단순 html을 React application으로 초기화하는 작업

---

## use client

위에서 이야기 하지 않았던 점이 있는데, hydration과정은 모든 component에 대해서 발생하지 않는다.

client에서 hydrate되고 interactive하게 되는 component는 오직 use client라는 지시어를 맨 위에 가지고 있는 component들 뿐이다.

이것이 의미하는 바는 use client를 상단에 넣지않으면 그 컴포넌트는 hydrate가 되지 않는다. 즉, react-component가 되지 않고 html으로 남아있다.

그래서, 우리가 use client를 넣는 것이다. 만약에 우리가 hydrate를 해줘야하는 페이지 use client를 안넣어줘도 알아서 에러가 뜨기에 상관없다.

use client는 client에서만 일어나는 것이 아닌 server에서 render되고 front에서 hydrate가 되는 것이다. use hydrate라고 생각하면 된다.

그리고 use client를 사용하지 않는 모든 컴포넌트를 server component라고 한다.
server component는 hydrate를 하지않기 때문에 js를 적게 다운받아도 되서 유저입장에서 좋다고 볼 수 있다.

여기까지가 SSR, Hydration, use client의 개념이다.

---

**잠깐!**

지원되지 않는 패턴: 서버 컴포넌트를 클라이언트 컴포넌트로 가져오기  
서버 컴포넌트를 클라이언트 컴포넌트로 import 할 수 없습니다.

지원되는 패턴: 서버 컴포넌트를 클라이언트 컴포넌트에 props로 전달  
서버 컴포넌트를 클라이언트 컴포넌트에 prop으로 전달할 수 있습니다.  
일반적인 패턴은 React children prop을 사용하여 클라이언트 컴포넌트에 "slot"을 만드는 것입니다.

---

## Layout

이번에는 NextJs의 Layout 시스템을 공부해볼 것이다.
layout을 사용하는 이유는 우리가 app을 만들 때, 재사용 요소가 있기 때문이다.

만약 about-us라는 페이지로 간다면 nextjs는 컴포넌트를 화면에 렌더링하지 않고, layout 컴포넌트에 있는 export된 디폴트 컴포넌트를 렌더링한다. 그리고 url을 확인하고 about-us를 렌더링해야된다는 것을 확인하고 렌더리을 한다.

```TSX
<Layout>
	<AboutUs />
</Layout>
```

나타내면 요러식으로

그렇기에 그냥 layout의 children위에 Navigation 컴포넌트를 넣어주면 된다.

여기까지가 layout 시스템의 전반적인 내용이였고, 이제 몇 가지 특별한 점에 대해서 알아볼 것이다.
예를 들어 about-us 페이지를 위한 layout를 만들수도 있다.

about-us 폴더안에 layout을 만들어줄 수 있다.
만약에 about-us 폴더에 layout을 만들었는데 about-us안에 더 깊은 depth의 폴더가 있다면 그 페이지도 about-us의 layout이 중복 적용이 된다.

> layout은 중첩이 되는 것이지 대체가 되는 것이 아님

## Route Group

route group은 routes들을 그룹화해서 다양한 것들을 할 수 있다.
루프 레이아웃을 사용하지 않고 대신 여러 레이아웃을 사용한다던지 레이아웃을 선택하고 선택 해제하여 사용할 수 있다.

사용해보기 위해서는 root에는 layout과 not-found는 전체에 적용되는 것이니 놔두고, 개별적인 존재인 page를 (home)이라는 폴더를 만들어 안에 넣어준다.

route group은 폴더 이름을 괄호롤 묶어줘야한다.
괄호로 묶어주게 되면 URL을 생성하지 않는다.

이것에 대해서는 차후에 자세히 알아봐야 할 것 같다.

## MetaData

> Next.js에는 향상된 SEO 및 웹 공유성을 위해 애플리케이션 메타데이터(ex: HTML head 엘리먼트 내의 meta 및 link 태그)를 정의하는 데 사용할 수 있는 메타데이터 API가 있습니다.

html의 head라고 생각하면 된다.

page와 layout에서만 metadata를 내보낼 수 있다.
그리고 metadata는 컴포넌트에서는 내보낼 수 없고 서버 컴포넌트에서만 있을 수 있다.

metadata에는 다양한 property가 있기에 찾아볼 것

## Dynamic Routes

Dynamic Routes는 Static과 다르게 url의 뒤쪽이 자유롭게 바뀌는 Route이다.
사용하기 위해서는 Movie라는 폴더가 존재한다면 Movie/\[id] 처럼 대괄호로 감싸서 만들어주면 된다.

그러면 뒤에 어떤 값을 넣어도 이동하는 것을 볼 수 있다.

그러면 뒤에 값을 어떻게 가져올 수 있을까?
props를 출력해보면Params와 searchParams라는 값을 넘겨준다.

react router에서 `movie/:id`처럼 쓰는것과 같다고 볼 수 있다.

> 동적 세그먼트는 폴더 이름을 대괄호(\[folderName])로 묶어 생성할 수 있습니다.

끝

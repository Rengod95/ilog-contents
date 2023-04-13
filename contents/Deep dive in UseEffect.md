---
title: Deep dive in UseEffect
date: 2023/02/27
author: 이인
tags:
  [
    "JavaScript",
    "TypeScript",
    "useEffect",
    "side effect",
    "scope",
    "dependency",
    "React",
  ]
---

# 왜 useEffect가 필요한가?

## 리액트에서 side effect 란 무엇을 의미하는가

리액트 공식문서에 따르면 useEffect 는 함수형 컴포넌트에서 발생할 수 있는 side effect를 제어하기 위한 훅이다.

따라서 useEffect를 살펴보기 전에, 우리는 먼저 프로그래밍에서의 사전적 의미의 side effect 가 무엇인지 알고 넘어가야할 필요가 있다.

프로그래밍에서, side effect는 함수나 연산이 자신의 범위(scope) 외부에 있는 상태(state)를 변경하는 모든 변화를 의미합니다. 이러한 상태는 전역 변수(global variable)를 수정하거나, 외부 데이터 구조(external data structure)를 수정하거나, 다른 함수의 상태(state)를 변경하는 것을 포함합니다.

side effect의 정의를 리액트의 문맥으로 치환해서 이해해보자. 가장 중요한 말은 scope 이다. 리액트의 side effect란 리액트가 통제하고 있는 것 외의 모든 것이라고 치환할 수 있다.

즉 리액트가 관여하는 component와 life cycle 시스템 외부 요인들을 의미한다. 예를 들어 DOM 을 직접 조작하려고 시도하거나, API 호출을 통해 외부 데이터를 패칭하는 것, 타이머나 이벤트 리스너를 세팅하는 것 등이 모두 포함된다고 볼 수 있다.

## 왜 우리가 side effect를 신경써야 하나

그렇다면 왜 우리가 side effect를 신경써야 하는 것인가?

수많은 상황이 있겠지만 가장 빠르게 이해할 수 있는 것은 API 호출이다.

특정 컴포넌트 내부에서 axios와 같은 데이터 패칭 도구를 통해 API를 호출하여 페칭된 데이터를 기반으로 화면에 출력을 해 주어야 하는 상황을 생각해보자. 이때 발생할 수 있는 변수와 상황은 너무 많다.

예로 데이터 페칭 도중 네트워크 연결이 끊긴다거나, 잘못된 요청으로 인해 정상적인 데이터를 받아오지 못했다거나, 한 번에 여러개의 데이터를 패칭했을 때 내가 의도한 순서와 다르게 응답이 온다거나 등.

그렇다면 axios가 이러한 상황을 모두 고려하여 동작해야 하는가? 리액트의 생명주기를 신경쓰는가? 언제 컴포넌트가 마운트되고 업데이트 되고 언마운트 되는지를 고려하여 데이터를 패칭하는가?

아니다. 관심사가 다르고 목적이 다르다.

데이터 패칭이라는 행위는, 리액트에 대한 아무런 종속이 존재하지 않고 독립적으로 동작해야 하는 행위이다. 즉 컴포넌트가 렌더링되는 과정을 외부 시스템(데이터 패칭)이 고려할 수도 없고, 고려해서도 안된다.

즉 리액트의 process에 아무런 관심을 가지지 않는다는 말이다.

이 때 리액트 입장에서 데이터 패칭은 리액트 자체적으로 제어할 수 없는 외부 영역이다. 이를 external system이라고 한다.

그러나 우리의 목적은 다르다. 패칭된 데이터를 기반으로 화면에 데이터를 출력해 주어야 한다. 리액트를 활용해서!

그렇다면 이 external system의 상태와 리액트의 상태를 동기화 하는 작업이 필요하다! 쉽게 말하자면, 사이드 이펙트를 의도에 맞게 제어할 수 있어야 한다! (여기서 말하는 상태란 포괄적인 의미이다. 외부시스템과 컴포넌트 모두 같은 목적을 가지고 동일한 시점에 대해 작업을 수행해야 한다는 말이다.)

그러면 외부 시스템과 리액트의 상태를 동기화, 제어하는 것을 누가 해주어야 하는가? 리액트? 아니다.

바로 코드를 작성하는 우리가 해 주어야 한다. 각자의 목적이 다르기 떄문이다.

다시 상기해 보자. 간단하게 리액트의 목적은 props 와 state에 맞는 컴포넌트의 상태를 정의하고 화면에 렌더링하는 것이다. 또한 axios와 같은 도구는 그저 사용자의 요청에 맞는 데이터를 패칭하는 것이 목적이다. 그리고 **우리의 목적은 리액트와 axios 모두를 사용하여 원하는 화면을 구성하는 것이다.**

결국 리액트의 입장에서 side effect를 발생시키고 싶은건 코드를 작성하는 '우리'이기 때문에, side effect를 리액트의 시스템에 맞게 통제하고 동기화 하는 것은 사용자인 우리의 몫이다. 여기서 useEffect의 의미를 이해할 수 있다.

# The useEffect

## useEffect 의 선언적 특성

useEffect는 컴포넌트 내부에서 사용자가 발생시키는 side effect를 제어하기 위한 훅이다.

아주 친절하게도 리액트는, useEffect는 side effect를 관리할 코드를 작성만 해 주면, component life cycle에 맞추어 알아서 코드를 실행해 준다. 우리가 일일이 리액트의 상태와 외부 시스템의 상태를 고려하는 것이 아니라, 단지 각 단계에 맞는 함수만 useEffect에게 전달 해 준다면 리액트가 알아서 컴포넌트의 life cycle에 맞게 컴포넌트와 외부 시스템을 동기화 시켜준다.

선언형의 관점이 잘 들어나는 부분이라고 볼 수도 있다. 컴포넌트의 라이프 사이클에 대한 구체적인 이해가 필요없이 각 상황별로 실행할 코드만 작성해주면 알아서 리액트가 관리해 주니 말이다.

단지 우리가 짚고 넘어가야 할 부분은 useEffect 훅은 우리가 이런 life cycle 을 신경쓰지 않도록 하는 대신에, side effect를 발생시키는 코드의 실행 주체를 리액트가 담당하게 된다.

우리가 원하는 부분에서 직접 호출하는 것이 아니라, 함수만 전달하고, 함수의 실행은 리액트가 자체적으로 원하는 시기에 실행 해버리니 말이다.

## useEffect 처럼 생각하기

처음 useEffect를 배우거나, 혹은 클래스형 컴포넌트의 작성방식에 익숙한 사람일 경우, 컴포넌트의 구체적인 life cycle과 엮어서 많이 생각하곤 한다.

예로 "컴포넌트가 렌더링 되고 나서 xx을 실행해야 하니까, 이 시점에서 데이터를 패칭해야 하고, 이 시점에서는 컴포넌트가 unmounted 될 것 같으니 이때는 yy를 수행해야 겠다." 혹은 "componentDidMount가 어느 시점이고 해당 시점에서 ~~ 함수를 수행하고, componentDidUnmount가 언제...." 이런 식으로 말이다.

즉 side effect를 컴포넌트의 렌더링이나 이벤트에 따른 콜백의 의미로 받아들이곤 한다. 해당 방식이 틀렸다고 단정짓는 것이 아니라, useEffect 훅이 등장한 시점에서 이런 사고방식이 과연 효율적인지를 고민해봐야 한다.

선언형 프로그래밍을 중시하는 리액트의 관점과, useEffect 훅의 사용 방식을 고려해 보았을 때, 개발자들의 의도는 '컴포넌트 내부에서 발생하는 side effect를 선언적으로 제어하도록 도와줄게!' 이다.

이 말은 곧 "너는 컴포넌트의 렌더링이나 이벤트 발생 타이밍을 신경쓸 필요가 없어, 단지 각 단계별로 어떻게 동작해야 하는지만 정의해 주었으면 해!" 로 치환할 수 있다. 리액트 공식문서에서는 이를 'useEffect 처럼 생각하기' 라고 명명하기도 한다.

이제 우리는 조금 더 효율적으로 사고하기 위해, 즉 useEffect처럼 생각하기 위해서 기존의 사고방식을 조금 바꿔볼 필요가 있다. 이를 위해 컴포넌트의 구체적인 라이프 사이클에 대한 고민보다는 간단하게 세 단계로 컴포넌트의 상태를 나타내보자.

1. mounted (컴포넌트가 화면에 나타났을 때)
2. updated (화면에 존재하던 컴포넌트가 내부 상태값의 변경으로 리렌더링 되었을 때)
3. unmounted (화면에 존재하던 컴포넌트가 사라졌을 때)

세 단계는 매우 직관적이다. 내부적으로 어떻게 각각의 프로세스를 분류하는지는 중요하지 않다. 단지 각각의 프로세스가 어떤 상태인지 집중하자.

useEffect는 이 직관적인 세 단계 마다 각각 실행해야 하는 함수를 받는다.

리액트 공식문서의 예시를 보자
https://codesandbox.io/s/oyyqps?file=%2FApp.js&utm_medium=sandpack

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
```

위 코드를 useEffect의 관점으로 생각해서 정리해보면 다음과 같다.

다음의 단계를 순차적으로 수행한다.

1.  `ChatRoom` mounted with `roomId` set to `"general"`
2.  `ChatRoom` updated with `roomId` set to `"travel"`
3.  `ChatRoom` updated with `roomId` set to `"music"`
4.  `ChatRoom` unmounted

이 때 effect의 관점에서 각 단계에 대한 effect의 수행 제어이다.

1.  Your Effect connected to the `"general"` room
2.  Your Effect disconnected from the `"general"` room and connected to the `"travel"` room
3.  Your Effect disconnected from the `"travel"` room and connected to the `"music"` room
4.  Your Effect disconnected from the `"music"` room

얼마나 직관적인가, mounted 될 때는 전달한 함수에서 setup 부분만 실행하고,
updated 될 때는 clean-up 함수 - setup 함수를 순차적으로 실행하고,
unmounted 될 때는 clean-up만 수행한다.

컴포넌트의 입장에서 생각하는 것은 더 복잡한 사고를 유발할 수도 있다. 언제 컴포넌트가 mounted, unmounted, updated 되는지는 중요하지 않다. 단지 각 단계의 컴포넌트 상태가 어떤 것인지를 이해하고, 각각의 상태에 대해 어떤 방식으로 외부 시스템과 컴포넌트를 동기화 할 것인가를 더 중점적으로 고민하고 작성해야 한다.

## 왜 dependency가 필요 할까

단순하게 생각해서 side effect를 컴포넌트가 화면에 보여질 때, 사라질 때의 두 가지 상태에 따라 제어하도록 할 수도 있다. 그러나 실질적인 컴포넌트의 상황은 그리 단순하지가 않다.

예로 채팅방 컴포넌트를 생각해 보자. 카카오톡의 채팅방 UI는 단 하나이다. 그저 누구랑 대화 하냐에 따라서 보여지는 채팅 목록이나, 대상의 프로필이 달라지는 등의 역할이 있다.

각 상대방 마다 모두 다른 채팅방 컴포넌트를 생성하기는 매우 비효율적이다. 단순하게 생각했을 때, 그냥 동일한 컴포넌트에서 보여지는 데이터만 달라지도록 로직을 작성하는 것이 재사용적인 측면에서나, 자원 측면에서나 훨씬 효율적이다.

예를 들어 채팅방에서 보여줘야 하는 데이터 중, 상대방의 프로필 이미지가 채팅방 컴포넌트로 전달되는 userId에 따라 달라져야 한다고 가정해보자.

userId는 props를 통해 부모에게서 주입받는다고 가정하면, 채팅방 컴포넌트 내부에서 side effect인 api 호출을 통해 userId를 기반으로 프로필 이미지를 불러와야 한다.

A유저와의 채팅방 컴포넌트가 mounted 되며 프로필 이미지를 불러오고난 후, 해당 채팅방 컴포넌트가 unmounted 되기 이전에 B유저와 대화하기 버튼을 누른다.

이 때 채팅방 컴포넌트는 이미 mounted 된 상태이기 때문에, 분명 B상대방과 대화하고 있음에도 불구하고, 프로필 이미지를 불러오는 side effect는 다시 실행되지 않는다.

즉 A유저의 프로필 이미지를 통해 컴포넌트가 작동한다. 채팅방 컴포넌트를 끄고 다시 켜야만 상대가 업데이트한 프로필 이미지가 제대로 반영이 될 것이다.

이런 상황을 방지하기 위해서는 userId가 변경됨에 따라 다시 side effect를 수행할 수 있어야 한다. 결국 mounted 이외에 updated 상황에 대해 side effect를 제어할 수 있어야 한다.

그래서 dependency가 필요하다. 같은 컴포넌트라도 특정 상태값이 바뀔 때 마다 다시 작동해야 하는 side effect를 제어할 수 있어야 한다. 이때 필요한 상태값들이 dependency로 들어가게 되는 것이다. 여기서는 userId라고 생각할 수 있다.

## dependency는 reactive value 여야 한다.

dependency가 왜 필요한지를 배우며 또 하나 알 수 있는건 dependency의 값, 즉 side effect가 실행되기 위해 필요한 종속성을 가져야 하는 값은 reactive value 여야 한다는 것이다.

reactive value는 직역하자면 반응형 값이다. 그렇다면 변할 수 있는 모든 값이 reactive value라고 생각할 수 있지만 그렇지 않다.

reactive value는 변화에 대응하여 스스로 업데이트할 수 있는 값을 의미한다.

이런 관점에서 props와 state는 당연히 reactive value라고 볼 수 있다. 또한 해당 값들에서 파생되는 모든 값들 또한 reactive value라고 볼 수 있다. props나 state에서 파생되는 값들은 결국 해당 값들에 대해 의존성을 가지며, 해당 값들에 대해 특정 로직을 수행한 결과값이기 때문이다. 그리고 이런 파생값들은 컴포넌트의 body 내에서 계산되는 값들이다.

결국 컴포넌트 내부(Body)에서 props나 state에서 파생된 값들이 존재한다면 그 값 또한 reactive value이다.

그렇다면 순수하게 컴포넌트가 관리하는 props와 state, 그리고 거기에서 파생되는 값들만 reactive value가 될 수 있는 것인가? 그렇지 않다. 전역 상태값을 생각해보자.

useEffect가 side effect를 수행할 때 의존성을 가지는 값이 login user id라고 생각해보자. 그리고 일반적인 상황에서 login user id는 보통 전역 상태값으로 관리하게 된다. 그리고 이 login user id는 값이 변함에 따라 컴포넌트가 자체적으로 리렌더링을 수행할 수 있는 값이다. 즉 리액트에서 자체적으로 변화를 감지하여 리렌더링을 수행할 수 있는 값이다.

반대로 모든 가변 값이 reactive value가 될 수 없다는 말과 동일하다. 예로 location.pathname 는 언제든지 변할 수 있는 가변 값이다. 그러나 location.pathname과 같은 mutable value는 리액트 상태관리 시스템이 통제하는 값이 아니라 그저 DOM API가 제공하는 location 객체의 프로퍼티 일 뿐이다.

따라서 컴포넌트 내부에서 location.pathname을 의존성으로 같는 useEffect 훅이 있다고 가정하면, 리액트는 location.pathname이 변하는 것을 감지하지 못합니다. 즉 location.pathname과 같은 리액트에 의해 통제되지 않는 mutable value를 side effect가 수행되는 의존성으로 두어서는 안된다는 것입니다.

같은 맥락으로, state가 아닌 외부 상수값들 또한 의존성으로 두어서는 안됩니다.

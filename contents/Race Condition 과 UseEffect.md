---
title: Race Condition 과 UseEffect
date: 2023/03/17
author: 이인
tags: ["JavaScript", "TypeScript", "useEffect", "Race Condition", "React"]
---

비동기 함수는 데이터 패칭이 완료되는 순서를 보장할 수 없다.
예로 1,2,3 번 데이터를 순차적으로 비동기 패칭을 진행하였을 때, 응답 데이터의 순서도 1,2,3 이라는 것을 보장할 수 없다는 말이다.
![](https://i.imgur.com/Fv7x5RP.png)
=> 사용자의 동작과 UI 간 불일치가 발생할 수 있다.

일반적으로 리액트는 사용자가 발생시키는 이벤트의 순서에 별 신경을 쓰지 않는다. 그저 상태 업데이트 예약이 발생하면 그에 맞게 리렌더링을 진행할 뿐이다.

그래서 별도의 조치가 없었다면, 일반적으로 짧은 시간 내에 동일 UI를 그려내는 데이터를 다른 파라미터를 기반으로 여러번 호출할 경우, 어떤 데이터가 먼저 호출되었냐에 관계 없이, 데이터가 먼저 전달되는 순서대로 상태값에 반영되는 것이 일반적이다.

그러나 이러한 상황은 의도된 것이 아니다. 사용자의 입장에서, 서로 다른 버튼을 여러번 빠르게 눌렀다고 해도, 화면에 나타나야 하는 것은 마지막 버튼 클릭에 대한 화면이어야 한다.

개발자의 입장에서도 이런 상황을 의도한 것이 아니기 때문에 명백히 버그라고 볼 수 있다.

간단히 정리하자면 데이터의 요청과 응답과정에서 발생하는 딜레이가 요청 데이터의 종류나 네트워크 상황에 따라 다르기 때문에, 이런 외부 데이터와 실제 사용자의 화면 간에 정상적인 동기화가 이루어지지 못한 것이다.

그렇다면 어떤 방식으로 이 문제에 접근해야 할까

1. API 호출에 대해 신경쓰지 않고, 컴포넌트 자체에서 연속된 데이터 요청에 대응하여 UI 처리를 진행한다.

```js
import React, { useEffect, useState } from "react";

export default function Post(props) {
  const [data, setData] = useState(null);
  let shouldRender = true;

  useEffect(() => {
    if (props.id == null) {
      setData(null);
      return;
    }

    const fetchData = async () => {
      if (props.id === 1) {
        await delay(2000);
      }
      const newData = await fetch("URL").then((response) => {
        console.log("데이터 패칭 완료", props.id);
        return response.json();
      });
      if (shouldRender) {
        setData(newData);
      }
    };

    fetchData();

    return () => {
      console.log(props.id, "캔슬");
      shouldRender = false;
    };
  }, [props.id]);

  if (data) {
    return (
      <div>
        <h2> postID: {data.id} </h2>
        <h1> {data.title} </h1>
        <p> {data.body} </p>
      </div>
    );
  } else {
    return <div>noting</div>;
  }
}

function delay(time) {
  return new Promise((resolve) => setTimeout(resolve, time));
}
```

![](https://i.imgur.com/Ms3yGCD.png)

2. 짧은 시간 내의 동일 종류의 데이터를 요청하는 API 호출 자체의 처리를 진행한다.

1번 방식의 문제점은 API가 언제 얼마나 호출되었는지에 관심을 두지 않기 때문에 연속된 이벤트에 대해 연속된 HTTP REQUEST가 생성된다. 즉 불필요한 통신 요청에 대한 예방이 되지 않는다.

2번 방식은 UI와 외부 데이터에 대한 동기화 뿐만 아니라, 애초에 연속적인 불필요한 통신 요청을 방지한다.

이때 필요한 도구가 바로 AbortController 이다.
https://developer.mozilla.org/en-US/docs/Web/API/AbortController

```js
import React, { useEffect, useState } from "react";

export default function Post(props) {
  const [data, setData] = useState(null);
  const abortController = new AbortController();

  useEffect(() => {
    if (props.id == null) {
      setData(null);
      return;
    }

    const fetchData = async () => {
      if (props.id === 1) {
        await delay(2000);
      }

      try {
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/posts/${props.id}/`,
          {
            signal: abortController.signal,
          }
        );
        const newData = await response.json();
        setData(newData);
      } catch (error) {
        if (error.name === "AbortError") {
          // do something
        }
      }
    };

    fetchData();

    return () => {
      abortController.abort();
    };
  }, [props.id]);

  if (data) {
    return (
      <div>
        <h2> postID: {data.id} </h2>
        <h1> {data.title} </h1>
        <p> {data.body} </p>
      </div>
    );
  } else {
    return <div>noting</div>;
  }
}

function delay(time) {
  return new Promise((resolve) => setTimeout(resolve, time));
}
```

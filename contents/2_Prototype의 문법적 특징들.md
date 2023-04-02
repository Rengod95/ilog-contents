자바스크립트의 객체는 명세서에서 명명한 `[[Prototype]]`이라는 숨김 프로퍼티를 갖습니다. 이 숨김 프로퍼티 값은 `null`이거나 다른 객체에 대한 참조가 되는데, 다른 객체를 참조하는 경우 참조 대상을 '프로토타입(prototype)'이라 부릅니다.

![[스크린샷 2023-02-02 오후 11.39.10.png]]

`__proto__`을 사용하면 값을 설정할 수 있습니다.

```js
let animal = { eats: true }; 
let rabbit = { jumps: true };
_rabbit.__proto__ = animal;
```

`__proto__`는 `[[Prototype]]`용 getter·setter입니다.

`__proto__`는 `[[Prototype]]`과 _다릅니다_. `__proto__`는 `[[Prototype]]`의 getter(획득자)이자 setter(설정자) 입니다.

하위 호환성 때문에 여전히 `__proto__`를 사용할 순 있지만 비교적 근래에 작성된 스크립트에선 `__proto__` 대신 함수 `Object.getPrototypeOf`나 `Object.setPrototypeOf`을 써서 프로토타입을 획득(get)하거나 설정(set)합니다.
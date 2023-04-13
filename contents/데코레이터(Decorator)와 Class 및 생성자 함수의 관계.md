---
title : 데코레이터(Decorator)와 Class 및 생성자 함수의 관계
date: 2023/03/27
author: 이인
tags: JavaScript, TypeScript, Decorator, Class, Prototype, HOC
---

# 데코레이터의 의미

Angular.js, nest.js 등과 같은 프레임워크의 주요 철학 중 하나가 바로 데코레이터이다. 데코레이터를 해당 프레임워크들이 어떻게 활용을 할까?

해당 프레임워크들의 플로우에 주요한 요소중 하나가 바로 메타데이터이다. 런타임에서 데코레이터를 통해 메타데이터가 주입된 객체들에 대해 프레임워크들이 평가를 진행하게 되고 메타데이터를 통해 각 객체의 역할이나 동작이 정해지게 된다.

예를 들어, `@Component` 데코레이터는 클래스가 Angular 컴포넌트임을 나타내는 메타데이터를 추가할 수 있다. 이러한 메타데이터는 런타임에서 Angular 프레임워크가 컴포넌트를 처리하고 렌더링하는 데 사용된다.

또 다른 예로는 `@Injectable` 데코레이터가 있다. 이 데코레이터는 클래스가 Angular 서비스임을 나타내는 메타데이터를 추가할 수 있다. 이러한 메타데이터는 런타임에서 Angular 프레임워크가 서비스를 처리하고 다른 구성 요소에서 사용할 수 있게 한다.

각 프레임워크가 사전에 제공해주는 데코레이터를 이용해 클래스의 동작을 임의로 정의하면, 실질적으로 해당 클래스의 동작을 변경할 수 있고 해당 작업을 프레임워크(외부)에 위임하게 되는 형태가 된다.

외부 프레임워크에 대해서만 의미를 가지는 것이 아니라 단일 데코레이터가 가지는 의미는 거대하다. 그리고 그 의미를 조금이나마 이해하기 위해 자바스크립트에서의 데코레이터가 무엇인지, 어떤 역할을 하는지, 어떻게 동작하는지 등을 이해해보자.

# 데코레이터란?
데코레이터(Decorators)는 클래스, 메서드, 프로퍼티, 파라미터 등에 부가적인 기능을 추가할 수 있도록 해주는 문법이다.

적어도 JS에서 데코레이터는 함수로 구현된다. higher-order-function 방식과 동일하게, 타겟을 함수로 감싸, 내부의 타겟이 참조할 수 있는 새로운 스코프를 만들어내고 이에 메타데이터 등을 추가하는 방식이다.

여기서 말하는 타겟은 클래스, 메서드, 프로퍼티, 파라미터 등을 말합니다. 정의된 데코레이터 함수는  `@` 기호를 사용하여 표시합니다. 예를 들어, 클래스에 데코레이터를 적용하려면 다음과 같은 구문을 사용할 수 있다.

```js
@decorator
class MyClass {
  // class implementation
}
```


# 데코레이터의 구현

가장 기본적인 형태의 데코레이터를 임의로 구현해 봅시다. JS에서 데코레이터 함수는 기본적으로 다음과 같은 형태로 구현된다.

```ts
function myDecorator(target: any, key: string, descriptor?: PropertyDescriptor) {
  // decorator implementation
}
```

해당 함수는 일반적인 형태의 데코레이터 함수로, 클래스의 선언부 및 구현부에 존재하는 클래스 멤버들을 대상으로 적용할 수 있다.

데코레이터의 각 argument가 어떤 객체를 레퍼런스 하는지를 이해하고, 사용되는 상황에 따라 다른 의미를 갖는다는 점을 이해하기 위해선 자바스크립트의 class 문법이 다른 언어와는 다르게 동작한다는 부분을 이해해야 한다.

# Class 문법의 진실

저번 글에서도 살펴봤다 시피 사실 자바스크립트에는 class라는 개념이 존재하지 않는다. 프로토타입을 기반으로 하기 때문이다.

그래서 class의 상속과 확장 등의 개념을 채용하기 위해서는 기존의 프로토타입 기반 시스템을 기반으로 새로운 class 문법을 구현해야 했고, 그 결과물이 현재의 class 문법이다. 이 때 가장 중심이 되는 기능이 자바스크립트의 생성자 함수이다.

생성자 함수를 살펴보기전 자바스크립트의 프로토타입 시스템에 대해 간단히 살펴보자. 

먼저 자바스크립트의 모든 객체는 프로토타입 객체가 존재한다. 그리고 이에 접근하기 위해서는 '[[prototype]]' 이라는 숨겨진 객체 프로퍼티가 존재하고 해당 객체를 통해 스코프 개념 및 프로토타입 체이닝 등이 가능하다. 해당 객체를 직접 get, set 하기 위해서는 __proto__  혹은  `Object.getPrototypeOf`나 `Object.setPrototypeOf` 등으로 접근할 수 있다.

일반적인 객체와 달라 생성자 함수는 [[prototype]] 과는 다른 prototype 이라는 프로퍼티를 가진다. 이 때, new 문법을 통해 생성자 함수를 이용한 객체를 생성했다면, 해당 객체는 자신을 생성한 생성자 함수의 prototype 을 [[prototype]] 으로 가지게 된다.

해당 개념을 통해 클래스 문법을 간단하게 생성자 함수와 prototype 기반으로 치환해 볼 수 있다.

```js
// 클래스 문법
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

// prototype과 생성자 함수로 치환한 코드
function Rectangle(width, height) {
  this.width = width;
  this.height = height;
}

Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
```

클래스의 constructor는 클래스의 이름을 가지는 생성자 함수의 구현부로 치환되며, prototype에 접근하여 특정 값을 할당하는 방식으로 클래스의 멤버를 정의할 수 있다.

여기서 클래스를 대상으로하는 일반적인 데코레이터 함수의 진짜 의미를 알 수 있다.
데코레이터는 클래스의 생성자, 메서드,프로퍼티 등에 메타데이터를 추가하거나, 작동 을 변경하는 것이 아니라, 사실상 생성자 함수의 [[prototype]]에 접근하여 해당 객체를 조작함으로써 작동 방식을 변경하는 것과 같다.

이제 데코레이터 함수의 각 argument가 의미하는 바를 다시 생각해보자.

- `target`  : 데코레이터가 적용되는 대상의 생성자 함수. 클래스 멤버에 적용할 때는 해당 멤버의 생성자 함수가 된다. 클래스를 대상으로 사용하던, 클래스의 멤버를 대상으로 사용하던, 결국 동일한 생성자 함수의 [[prototype]] 객체를 레퍼런스 하는 객체이다.
-   `key` :  데코레이터가 적용되는 대상의 이름이다. 클래스 멤버에 적용할 때는 해당 멤버의 이름이된다.
-   `descriptor` (선택적): 데코레이터가 적용되는 대상의 프로퍼티 디스크립터. 클래스 멤버에 적용할 때만 사용된다. [[prototype]] 객체의 각 프로퍼티가 가지는 값은 순수한 value 외에도 화면 밖에 숨겨진 정보들이 있는데, 이런 정보들이 각 프로퍼티가 어떻게 작동할지를 정의한다.

```js
console.log(Object.getOwnPropertyDescriptor(oatmeal, 'viscosity'));

/* 결과 :
{
  configurable: true,
  enumerable: true,
  value: 20,
  writable: true
}
*/
```

각 속성의 의미는 다음과 같다.
-   `구성 가능(configurable)`은 속성 유형을 변경하거나, 객체에서 속성을 삭제할 수 있는지를 결정한다.
-   `열거 가능(enumerable)`은 `Object.keys(oatmeal)`를 호출하거나 `for` 루프에서 사용할 때처럼 객체의 속성을 열거할 때 속성을 표시할지 여부를 제어한다.
-   `쓰기 가능(writable)`은 할당 연산자 `=`를 통해 속성값을 변경할 수 있는지를 제어한다.
-   `값(value)`은 접근할 때 표시되는 속성의 정적 값이다. 속성 설명자 중에 유일하게 쉽게 볼 수 있고, 주로 우리가 관심을 두고 보는 부분이다. 함수를 포함한 모든 자바스크립트의 값이 올 수 있으며, 이 속성은 속성을 자신이 속한 객체의 메소드로 만든다.

속성 설명자에는 다른  두 가지 속성 getter와 setter도 존재한다.

-   `get`은 정적인 `value` 대신 반환 값을 전달하는 함수이다.
-   `set`은 속성에 값을 할당할 때, 등호 오른쪽에 넣는 모든 것을 인자로 전달하는 특수 함수이다.


# 함수 대상의 데코레이터

데코레이터 문법은 원래 일반적인 함수를 대상으로 하는 것이 아니다. 기존에 구현된 함수의 구현을 수정하지 않고 새로운 내용을 추가하고 싶다면 대안이 많고 그 중 하나가 HOF(Higher-Order-Function)이다. 

결국 데코레이터라는 문법은 결과적으로는 클래스의 동작을 변경하지만 사실 데코레이터라는 래퍼 함수를 통해 참조할 수 있는 새로운 스코프를 만들어 내고 해당 스코프를 이용해 프로토타입 객체를 조작하는 방식이다.

HOF 도 일부 동일하다. 래퍼함수를 통해 새로운 스코프를 만들어 내어 기존 함수의 동작을 변경한다.

그렇다면 함수에 새로운 기능을 추가하고 싶다면 어떤 패턴을 사용하는게 적합할까?함수 또한 객체이니까 데코레이터를 사용해도 되지 않을까?

그렇지 않다.

특정 함수, 특정 객체는 결국 생성자 함수로 생성되는 건 동일하지만, 함수를 생성하는 생성자 함수는 Function 이다. 그리고 Function 객체는 일반적인 객체와는 다르게 생성자 함수를 가지고 있지 않습니다. 즉 일반적인 형태의 데코레이터 함수가 정상적으로 작동하지 않을 수 있다.

다음 코드를 통해 실제로 확인해보자.

```ts
function log(target: any, key: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function(...args: any[]) {
    console.log(`Method '${key}' called with arguments: ${JSON.stringify(args)}`);
    const result = originalMethod.apply(this, args);
    console.log(`Method '${key}' returned: ${JSON.stringify(result)}`);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(x: number, y: number): number {
    return x + y;
  }
}

const calc = new Calculator();
calc.add(1, 2);
// Output:
// Method 'add' called with arguments: [1,2]
// Method 'add' returned: 3

```

Calculator 라는 클래스의 클래스 멤버인 add에 데코레이터를 적용한 모습이다.
이 때 descriptor가 [[prototype]] 객체의 add 프로퍼티에 관한 정보를 가지기 때문에 정상적으로 참조하여 add 메서드의 동작을 변경할 수 있었다.

이제 해당 데코레이터를 함수에 적용해 보자.

```ts
@log
function multiply(x: number, y: number): number {
  return x * y;
}

multiply(2, 3);

//Uncaught TypeError: Cannot read property 'value' of undefined at log (app.ts:2) at app.ts:9
```

보다시피 value를 참조하지 못한다.

multiply 함수의 생성자함수인 Function의 prototype 객체에는 자바스크립트의 함수가 사용할 수 있는 공통된 프로퍼티들이 정의되어 있다. 

이 때 descriptor는 Function.prototype 의 multiply 라는 아이템을 참조하려고 시도할 것이다. 당연히 Function.prototype에는 multiply 라는 프로퍼티가 존재하지 않기 때문에 레퍼런스 에러가 발생한다.


위 예시를 통해 일반적인 데코레이터는 클래스와 클레스 멤버에 대해서 사용하는 것이 안전하다고 결론낼 수 있다.

multiply와 같은 함수의 경우 데코레이터 대신 같은 역할을 해줄 수 있으면서 함수의 동작 변경이라는 의도에 맞는 HOF가 적합한 패턴이라고 볼 수 있겠다.

그러나 특별한 상황이나 여러 조건들로 인해 함수에도 무조건 데코레이터를 사용해야 하는 상황이 발생할 수 있다.

이때 사용할 수 있는 방법은 target만을 데코레이터의 인자로 지정하는것이다.

```ts
function log(target: Function) {
  const originalFunction = target;

  const newFunction = function(...args: any[]) {
    console.log(`Function '${originalFunction.name}' called with arguments: ${JSON.stringify(args)}`);
    const result = originalFunction.apply(this, args);
    console.log(`Function '${originalFunction.name}' returned: ${JSON.stringify(result)}`);
    return result;
  };

  return newFunction;
}

```

target의 타입이 Function으로 고정된 것을 확인할 수 있다. 근데 위 코드는 우리가 어디서 많이 보던 패턴과 유사하다. 맞다 바로 HOF 방식과 일치한다. 즉 데코레이터를 함수를 대상으로 사용하려고 시도하면 결국 HOF의 형태로 작성될 수 밖에 없다.다만 @을 통해 가시성이나 코드 작성에 대한 편리한 부분 등의 장점을 취할 수 있다.
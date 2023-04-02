# 함수 호출 방식에 따른 this

자바스크립트의 함수는 호출될 때, 매개변수로 전달되는 인자값 이외에, [arguments 객체](https://poiemaweb.com/js-function#61-arguments-%ED%94%84%EB%A1%9C%ED%8D%BC%ED%8B%B0)와 `this`를 암묵적으로 전달 받는다.

 결국 자바스크립트의 함수는, 호출 방식에 의해 `this`에 바인딩할 어떤 객체가 동적으로 결정된다. 다시 말해, 함수를 **선언할 때 this에 바인딩할 객체가 정적으로 결정되는 것이 아니고,** **함수를 호출할 때 함수가 어떻게 호출되었는지에 따라** this에 바인딩할 객체가 동적으로 결정된다.

함수의 호출하는 방식은 결국 함수를 호출하는 주체가 누구냐에 따라 달라진다는 것과 같다. 결국에는 함수가 가지는 this는 함수를 호출하는 주체 즉, 발화(Invoke)의 주체가 누구냐에 따라 달라지게 된다.

다음은 발화 주체에 따른 호출 방식의 차이이다.

1.  함수 호출 - 발화 주체가 명시되지 않은 채 단독으로 호출된다.
2.  메소드 호출 - 특정 객체를 Dot를 통해 참조하여 호출된다.
3.  생성자 함수 호출 - new 키워드와 함께 호출된다.
4.  apply/call/bind 호출 - 발화 주체를 명시적으로 지정할 수 있다.


## 1. 일반 함수 호출 방식

여기서 말하는 일반 함수란, 특정 객체에 속하는 메소드로서의 호출이 아닌 전역 스코프에서 선언된 함수를 뜻한다.

일반적인 함수의 호출은 전역 객체 위에서 함수를 호출하는 것과 같다. 즉 발화의 주체가 전역객체이다. 전역객체(Global Object)는 모든 객체의 유일한 최상위 객체를 의미한다. 또 전역 객체는 코드가 실행되는 런타임 환경이 브라우저냐 Node.js냐에 따라 window 혹은 global 객체로 매핑된다.

즉 발화의 주체가 전역객체 이기 때문에 일반 함수의 this는 전역 객체로 바인딩 된다.


## 2. 중첩 함수의 this

**내부함수는 일반 함수, 메소드, 콜백함수 어디에서 선언되었든 관게없이 this는 전역객체를 바인딩한다.** 더글라스 크락포드는 “이것은 설계 단계의 결함으로 메소드가 내부함수를 사용하여 자신의 작업을 돕게 할 수 없다는 것을 의미한다” 라고 말한다. 

즉 함수 내에 중첩되어 선언된 모든 함수들은 발화의 주체가 전역 객체라고 인식되는 것이다.

```js
globalThis.name = "전역 객체 이름";

  

function sum(a, b) {
	console.log(this.name);
	return a + b;
}

function nestingWrapper(cb) {
	function wrapper(...args) {
		return cb(...args);
	}
	return wrapper;
}

const wrapped_sum = nestingWrapper(sum);
wrapped_sum(1, 2); // 전역 객체 이름

const util = {
	name: "유틸 객체 이름",
	sum: function wrapper(...args) {
			function sum(a, b) {
				console.log(this.name);
				return a + b;
			}
		return sum(...args);
		},
};
util.sum(3, 4); // 전역 객체 이름

```

위 코드를 통해 중첩함수의 this가 어떤 상황에서도 항상 전역 객체를 참조한다는 사실을 알 수 있다. 그러나 사용자는 중첩함수의 this가 상위 컨텍스트의 this를 참조하기를 원하는 상황이 더욱 많을것이다.

그렇다면 중첩 함수의 this가 어떻게 상위 컨텍스트의 this를 참조하도록 유도할 수 있는지 알아보자.


### 2-1. 명시적으로 this 바인딩

**util.sum(3.4)** 의 정의에서 반환값에 대해 apply나 call 이용해 this가 바인딩할 객체를 명시적으로 지정할 수 있다.

```js
const util = {
	name: "유틸 객체 이름",
	sum: function wrapper(...args) {
			function sum(a, b) {
				console.log(this.name);
				return a + b;
			}
		return sum.apply(util, args); 
		// sum이 호출될 때의 this는 항상 util객체를 가리킨다.
		},
};

util.sum(3, 4); // 유틸 객체 이름
```

### 2-2. 상위 컨텍스트의 this 캐싱

중첩 함수의 상위 컨텍스트가 형성되는 함수의 내부에서 this를 캐싱하여 중첩함수에서 사용할 수 있다.


```js
const util = {
	name: "유틸 객체 이름",
	sum: function wrapper(...args) {
			const cachedThis = this;
			function sum(a, b) {
				console.log(cachedThis.name);
				return a + b;
			}
		return sum(..args)
		},
};

util.sum(3, 4); // 유틸 객체 이름
```


## 3. 메소드로서의 호출

일반 함수의 경우 전역 객체에서 선언된 함수로써, 전역 객체가 발화의 주체로 해당 함수가 실행되는 경우였다. 메소드로서 함수를 호출한다는 말은, 객체의 프로퍼티로서 함수가 선언되어 있으며, 발화의 주체가 되는 객체가 누군지에 따라 this가 바인딩하는 객체가 달라지게 된다.

```js
const Person = {
	name: "사람 이름",
	getName: function () {
		return this.name;
	},
};

  

const Car = {
	name: "자동차",
};

Car.getName = Person.getName;

console.log(Person.getName()); // 사람 이름
console.log(Car.getName()); // 자동차
```


## 3. 생성자 함수로서의 호출

자바스크립트로 작성된 모든 함수는, new 키워드와 함께 호출되면 생성자 함수로서 동작한다. 일반적인 객체지향 문법의 생성자 함수와 비슷한 역할을 하지만 작동 방식은 다르다.

다음은 특정 함수가 생성자 함수로서 호출이 된 상황에 발생하는 과정이다.

**1. 빈 객체 생성 및 this 바인딩**
 - 생성자 함수는 새로운 빈 객체를 생성하는데 이 객체는 this로 바인딩 된다. 따라서 이후 생성자 함수 코드 내부의 this는 이 빈 객체를 가리킨다.
 - 이 새로운 객체는 자신을 생성한 **생성자 함수의 prototype 프로퍼티가 가리키는 객체를 자신의 프로토타입 링크로 설정한다.

**1-1. 프로토타입 프로퍼티**
-   **함수 정의 시, 함수와 동시에 생성되는 객체**
-   constructor 프로퍼티 존재. 동시에 __proto__ 프로퍼티 존재
-   생성된 **함수는 prototype 프로퍼티로 이 프로토타입 오브젝트를 참조**
-   **constructor 프로퍼티로 생성된 함수를 참조**

A라는 함수에 대해, A.prototype 을 통해 A 함수의 프로토타입 오브젝트에 접근 하며, A 함수의 프로토타입 오브젝트는 constructor 프로퍼티를 통해 자신의 원형 함수인 A함수를 참조하게 된다.

**1-2. 프로토타입  인터널 슬롯(internal slot)**
* 모든 객체가 소유하며, 자신의 프로토타입 객체를 가리킨다.
-   함수를 포함한 **모든 객체**가 가진다.
-   부모 역할을 하는 객체를 __proto__ 프로퍼티를 통해 참조 하여 부모 객체의 속성에 접근 가능
-   부모에게도 찾는 프로퍼티가 없으면 __proto__를 통해 최상위 계층까지 탐색한다.

**2. this를 통한 프로퍼티 생성**
생성된 빈 객체에 this를 사용하여 동적으로 프로퍼티나 메소드를 생성할 수 있다. this는 새로 생성된 객체를 가리키므로 this를 통해 생성한 프로퍼티와 메소드는 새로 생성된 객체에 추가된다.

**3. 생성된 객체 반환**
-   반환문이 없는 경우, this에 바인딩된 새로 생성한 객체가 반환된다. 명시적으로 this를 반환하여도 결과는 같다.
-   반환문이 this가 아닌 다른 객체를 명시적으로 반환하는 경우, this가 아닌 해당 객체가 반환된다. 이때 this를 반환하지 않은 함수는 생성자 함수로서의 역할을 수행하지 못한다. 따라서 생성자 함수는 반환문을 명시적으로 사용하지 않는다.

```js
function Person(name) {
	// 생성자 함수 코드 실행 전 -------- 1 
	this.name = name; // --------- 2
	// 생성된 함수 반환 -------------- 3 
} 
const me = new Person('Lee'); 
console.log(me.name);

```

-   객체 리터럴 방식의 경우, 생성된 객체의 프로토타입 객체는 Object.prototype이다.
-   생성자 함수 방식의 경우, 생성된 객체의 프로토타입 객체는 Person.prototype이다.
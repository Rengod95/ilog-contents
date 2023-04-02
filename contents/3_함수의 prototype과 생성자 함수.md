```js
function Person() {
	let pName = "private name"; 
	// this 키워드가 붙어있지 않은 모든 변수, 함수 는 private
	this.name = "test name";\
	// this가 붙어있다면 인스턴스를 통해 접근 가능한 public
	this.setPname = (nName) => {
		pName = nName;
	};
	this.getPname = () => pName;
}

const p1 = new Person();
const p2 = new Person();

p2.name = "modified";

console.log(p2.name);
console.log(p1.pName); // undefined
console.log(p1.getPname()); // private name

p1.setPname("modified p name");

// prviate, public 변수 모두 각각 다른 컨텍스트를 가진다. 모두 개별 인스턴스화
console.log(p1.getPname()); // modified p name
console.log(p2.getPname()); // private name

// 인스턴스에 새로운 프로퍼티를 추가하여 사용할 수 있다.
p1.getSpecialName = () => {
	console.log("스페셜 이름");
};

// 그러나 생성자 함수에 새로운 프로퍼티를 추가한다고 해서 인스턴스에 반영되지는 않는다.
Person.getSpecialName = () => {
console.log("스페셜 이름");
};


p2.getSpecialName(); // TypeError: p2.getSpecialName is not a function
```

생성자 함수(`F`)의 프로토타입을 의미하는 `F.prototype`에서 `"prototype"`은 `F`에 정의된 일반 프로퍼티라는 점에 주의해 글을 읽어 주시기 바랍니다. `F.prototype`에서 `"prototype"`은 바로 앞에서 배운 '프로토타입’과 비슷하게 들리겠지만 이름만 같을 뿐 실제론 다른 우리가 익히 알고있는 일반적인 프로퍼티입니다.
![[스크린샷 2023-02-02 오후 11.44.51.png]]

`F.prototype`은 `new F`를 호출할 때만 사용됩니다.

`F.prototype` 프로퍼티는 `new F`를 호출할 때만 사용됩니다. `new F`를 호출할 때 만들어지는 새로운 객체의 `[[Prototype]]`을 할당해 주죠.

새로운 객체가 만들어진 후에 `F.prototype` 프로퍼티가 바뀌면(`F.prototype = <another object>`) `new F`를 호출해 만드는 또 다른 새로운 객체는 another object를 `[[Prototype]]`으로 갖게 됩니다. 다만, 기존에 있던 객체의 `[[Prototype]]`은 그대로 유지됩니다.
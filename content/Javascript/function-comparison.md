---
title: function vs. arrow function
description: function-comparison
tags:
  - Javascript
date: 2026-01-28
draft: false
---
# 함수 선언 방식
JavaScript에서 대표적으로 함수를 선언하는 방법은 **function 키워드**와 **화살표 함수**이다.
두 방식은 문법뿐 아니라 동작 방식에서도 차이가 존재하기 때문에, 상황에 따라 적절히 선택하는 것이 중요하다.

## function

```javascript
function add(a, b) {
	return a + b;
};
```

- 호이스팅 (`hoisting`)이 적용되어, 함수 선언 이전에도 호출이 가능하다.
- 함수 내부에서의 `this`는 호출한 주체에 따라 동적으로 결정된다.
- `arguments` 객체를 사용할 수 있다.

## Arrow Function

```javascript
const add = (a, b) => {
	return a + b;
};

const. add = (a, b) => a + b;
```
- 호이스팅 (`hoisting`)이 적용되지 않는다.
- `this`를 새로 생성하지 않고, 상위 스코프의 `this`를 그대로 사용한다.
- `arguments` 객체를 사용할 수 없다.

## 차이점
### `this`

- `function` 함수 에서는 `this`는 누가 호출했는지에 따라 결정이 된다.
- 여기서는 `function` 으로 선언된 `getValue`가 호출되는 시점에서 `this`가 결정된다.
	- 즉, `this === obj` 이다.
```javascript
const obj = {
	value: 10,
	getValue: function() {
		console.log(this.value)
	}
}

obj.getValue(); // 10
```

- `this`를 새로 만들지 않고, 상위 스코프의 `this`를 그대로 사용한다.
	- 보통 전역 스코프에서 실행되기 때문에 실제 `this`는 `window`이다.
```javascript
const obj = {
	value: 10,
	getValue: () => {
		console.log(this.value)
	}
}

obj.getValue(); // undefined
```

- 객체의 메서드를 화살표 함수로 만들면 `this`가 객체를 가리키지 않는다.
- 객체 메서드는 `function`, 콜백은 화살표 함수를 기본적으로 사용하는 것이 좋다.

```javascript
const obj = {
	value: 10,
	getValue: function() {
		setTimeout(() => {
			console.log(this.value)
		}, 100)
	}
}

obj.getValue(); // 10
```
- 아니면 위의 예제처럼 바깥 `function`의 `this -> obj`를 이용하여, 안쪽의 화살표 함수는 그 `this`를 그대로 사용하도록 할 수 있다.
- 이것이 바로 상위 스코프의 `this`를 사용하는 것.
---
title: Remember과 by의 상관관계
description: Remember-MutableState
tags:
  - Kotlin
date: 2026-01-20
draft: false
---
# 상태관리

## Recomposition
* Compose는 데이터가 변하면 해당 데이터를 사용하는 UI Composable을 다시 호출하여 화면을 업데이트한다, 이를 재구성 `Recomposition` 이라고 칭한다.
* 아무런 장치 없이 일반 변수만 사용하면, 함수가 다시 호출될 때마다 변수도 초기값으로 리셋되는 문제 발생, 이를 위해 상태관리가 필요하다.
## MutableState
* 변화를 관찰하는 상태 값
- 이 객체 안에 담긴 `value`가 변경되면, 해당 값을 사용하는 화면 부분을 `Composable` 자동으로 `Recomposition` 시킨다.
- 단순 변수가 아닌 관찰 가능한 상태이기 때문에, 사용자의 입력이나 이벤트에 따라 UI를 실시간으로 업데이트할 때 필수적으로 사용
## Remember
* `Recomposition`이 발생하면, `Composable`함수가 처음부터 재실행되는데, 이때 상태 값도 다시 초기화 될 수 있으므로 `Remember`가 필요하다.
* 함수가 다시 실행되더라도 `Remember`로 감싼 값은 초기화되지 않고, 이전 단계에서 저장했던 값을 그대로 반환.

즉, `MutableState`는 값이 변했음을 알리고 UI를 갱신하고, `Remember`는 값을 보존하는 것이다.

```kotlin
@Composable
fun Counter() {
	var count = remember { mutableStateOf(0) }
	
	Button(onClick = {count.value++}) {
		Text(text = "Count: ${count.value}")
	}
}
```

# Remember, by

## `=` vs. by

```kotlin
var count = remember { mutableStateOf(0) }
```

```kotlin
var count by remember { mutableStateOf(0) }
```

위의 2가지 방식의 차이는 무엇일까

### `=` 부등호 방식
* 타입은 `MutableState<Int>` 이며, 상태 객체를 직접 다룬다.
* `.value`로 실제 값에 접근이 가능하다
* 상태임이 명확

```kotlin {3, 5-6}
@Composable
fun Counter() {
	var count = remember { mutableStateOf(0) }
	
	Button(onClick = {count.value++}) {
		Text(text = count.value.toString())
	}
}
```

- 여기서 `count`는 값이 아니라 **상태 객체 (state)** 임을 인지해야한다.

### by 방식
- by는 `Kotlin`의 `property delegation`이다. 
	- 즉, 위임을 한다는 것!
* 타입은 `Int` 이며, `MutableState`는 숨겨진다.
* 접근은 `getValue` / `setValue` 를 통해 `MutableState.value`로 위임된다
```kotlin {3, 5-6}
@Composable
fun Counter() {
	var count by remember { mutableStateOf(0) }
	
	Button(onClick = {count++}) {
		Text(text = count.toString())
	}
}
```

여기서 `count` 라는 값을 읽고 쓰는 모든 순간은 내부적으로 **`MutableState.value`에 대한 접근**으로 변환된다.

개념적으로 다음과 같이 해석이 가능하다. (이해를 돕기 위함)
```kotlin
val state = remember { mutableStateOf(0) }

var count: Int
    get() = state.value
    set(newValue) {
        state.value = newValue
    }
```
- `count`에 대한 `getter` / `setter`를 생성
- 내부에서 `state.getValue(...)` / `state.setValue(...)`를 호출
- 결과적으로 `state.value`로 연결된다.

## 그럼 무엇을 쓸까?

 [[#`=` 부등호 방식|부등호 방식]]  
- 상태를 다른 Composable로 전달해야할 때
- 상태 자체를 명확히 드러내고 싶을 때

```kotlin
@Composable
fun CounterScreen() {
    val countState = remember { mutableStateOf(0) }

    CounterButton(countState)
    CounterText(countState)
}
```

```kotlin
@Composable
fun CounterButton(countState: MutableState<Int>) {
	Button(onClick = { countState.value++ }) {
		Text("Increase")
	}
}
```

```kotlin
@Composable
fun CounterText(countState: MutableState<Int>) {
    Text(text = "Count: ${countState.value}")
}
```

[[#by 방식|by 방식]]
- UI 코드
- 클릭, 입력 값 처리
- 화면에서 값처럼 쓰고 싶을 때

```kotlin
@Composable
fun SimpleCounter() {
	var count by remember { mutableStateOf(0) }
	
	Button(onClick = { count++ }) {
		Text(text = "Count: $count")
	}
}
```

```kotlin
fun NameInput() {
	var name by remember { mutableStateOf("") }
	
	TextField(
		value = name,
		onValueChange = { name = it }
	)
	
	Text(text = "Hello, $name")
}
```
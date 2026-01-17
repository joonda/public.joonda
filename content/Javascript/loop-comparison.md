---
title: forEach vs. for...of vs. for...in
description: loop-comparison
tags:
  - Javascript
date: 2025-05-19
draft: false
---

## 이걸 생각하게 된 이유
- 현재 프로젝트에서 유효성 검사를 진행하고 있는데, `Vue`단에서 입력 값을 확인하여 검사를 진행해야한다.
  그러다가 `forEach`, `for of`, `for in` 등의 반복문 등을 많이 접하게 되면서 검색을 해보았다.

## 상황
- 현재 [Tabulator](https://tabulator.info/docs/6.3) 라는 그리드 라이브러리를 사용하고 있다.
```javascript
selectedRows.forEach((row) => {
  if (!row.itemnm || row.itemnm.trim() === '') {
    vAlertError('품목명을 입력해주세요.');
    return;
  } else if (!row.unit || row.unit.trim() === '') {
    vAlertError('단위를 입력해주세요.');
    return;
  } else if (row.reqqty == null || row.reqqty <= 0) {
    vAlertError('요청수량을 입력해주세요.');
    return;
  }
});
```
- `selectedRows`는 `Tabulator`의 `Grid`에서 선택된 행 들의 배열이다.
- 처음에는 `selectedRows`를 `forEach`로 순회하여 각각의 항목을 검사하고, 유효하지 않으면 에러메시지를 띄우려 했다.
- 하지만 이렇게 하면 **오류가 떠도 반복문이 중단되지 않고** 다음 항목까지 계속적으로 검사하게 된다.
    - 즉 **루프제어가 불가능** 하다.
- 또한 `forEach`는 `async/await`와도 어울리지 않는다.
    - 비동기 함수는 순차적으로 처리되길 기대하지만, `forEach`는 async 콜백이 끝날 때까지 기다려주지 않고, 콜백들을 동시에 실행 시킨다.
    - 즉, `forEach`는 순차적 처리로 동작하지 않는다

## 해결법
```javascript
for (const row of selectedRows) {
  if (!row.itemnm || row.itemnm.trim() === '') {
    vAlertError('품목명을 입력해주세요.');
    return;
  } else if (!row.unit || row.unit.trim() === '') {
    vAlertError('단위를 입력해주세요.');
    return;
  } else if (row.reqqty == null || row.reqqty <= 0) {
    vAlertError('요청수량을 입력해주세요.');
    return;
  }
}
```
- 그래서 위 코드 처럼 `for...of`로 변경했다.
- for...of 는 배열, 문자열, Set, Map 등 `iterable` 객체를 순회할 수 있으며, `break`, `continue`, `return`과 같은 루프제어가 가능하다.
- **비동기 처리에 유리**하여 `async`, `await`도 자유롭게 사용할 수 있다.

## 정리
- `forEach`: 배열 값 순회 (중단 불가)
```javascript
const arr = ['a', 'b', 'c'];
arr.forEach((val) => {
  console.log(val);           // a, b, c
  // return, break 불가
});
```
- `for of`: 값(value)를 순회 (return, break, continue 등 루프 제어 가능)
```javascript
const arr = ['a', 'b', 'c'];

for (const val of arr) {
  console.log(val);           // a, b, c
}
```
- `for in`: 객체의 키를 순회 (return, break, continue 등 루프 제어 가능)
```javascript
const obj = { name: '철수', age: '30' };

for (const key in obj) {
  console.log(key);           // name, age
  console.log(obj[key]);      // 철수, 30
}
```
- `for in`은 배열에도 사용 가능하지만, **순서 보장되지 않고** 상속된 키도 순회할 수 있으므로 배열에는 **사용을 권장하지 않는다**.
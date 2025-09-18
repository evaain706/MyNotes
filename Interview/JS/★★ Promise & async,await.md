

## Promise

- 비동기 연산의 상태를 나타내는 ** 객체 **
- 비동기 처리가 진행중(pending),성공(fullfilled), 실패(rejected)라는 3개의 상태를 가짐
- 비동기 프로그래밍을 then() 과 catch() 체이닝을 통해 보다 간결하게 표현가능하도록 도입

### 예시
```js
const myPromise = new Promise((resolve, reject) => {
    const success = true; // 성공/실패 조건

    setTimeout(() => {
        if (success) {
            resolve("요청 성공!");
        } else {
            reject("요청 실패...");
        }
    }, 1000);
});

myPromise
    .then((res) => console.log(res))  // 성공 시 실행
    .catch((err) => console.log(err)) // 실패 시 실행
    .finally(() => console.log("완료!")); // 항상 실행

```


## Promise 등장 이전의 비동기처리는?

```js
function doSomething(Callback, errCallback) {
    setTimeout(() => {
        const success = Math.random() > 0.5;
        if (success) {
            console.log("작업 완료");
            Callback();
        } else {
            errCallback("작업 실패");
        }
    }, 1000);
}

doSomething(
    () => console.log("콜백 실행"),
    (err) => console.log(err)
);

```

#### Promise 등장 이전에는 주로 *콜백함수*를 사용한 비동기처리
- 가장 기본적인 비동기 처리방식
- 비동기 함수에 성공시 콜백함수, 실패시 콜백함수를 넘겨서 상태에 따른 처리를함
- 함수에 작업완료후에 실행할 함수를 전달하는 형태를가짐
- 작업이 많아지면 많아질수록 가독성과 유지보수성이 떨어지는 콜백지옥형성 가능성



## async-await란??

- Promise의 완료를 기다리기위한 문법
- 비동기 코드를 동기코드처럼 작성가능하게 해줌
- async키워드를 붙여 정의된 함수내에서 호출되는 Promise앞에 await 키워드를 붙임
   
 ```js
 async function getData() {
    try {
        const res = await fetch("https://example.com/data");
        // await가 붙은 부분이 완료될때까지 기다림
        const data = await res.json();
        console.log(data);
    } catch (err) {
        console.log("에러 발생:", err);
    }
}

getData();

 ```
 - async 함수는 *항상* Promise 객체를 반환 -> 반환값이 바로 Promise로 감싸짐
 - await 키워드가 붙은 Promise가 처리될때까지 기다린뒤 다음코드를 실행
 - await 사용시 try/catch 문법을 사용하여 동기코드와 같이 에러처리가 가능함
 
 
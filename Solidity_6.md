# Chapter 6 (Solidity Front)

## Web3.js Intro

- 이더리움 네트워크는 노드로 구성되어 있음
- 각 노드는 블록체인의 복사본을 가짐
- 스마트 컨트랙트의 함수를 실해앟고자 하면, 이 노드들 중 하나에 질의를 보내야함

> 질의를 보낼때 전달해야할 내용

1. 스마트 컨트랙트의 주소
2. 실행하고자 하는 함수
3. 함수에 전달하고자 하는 변수들

이더리움 노드들은 JSON-RPC라 불리는 언어로만 소통할 수 있음, 이는 사람이 읽기 불편, 아래는 질의의 예시

```ts
CryptoZombies.methods
  .createRandomZombie('Vitalik Nakamoto')
  .send({ from: '0xb60e8dd61c5d32be8058bb8eb970870f07233155', gas: '3000000' });
```

Web3.js 를 추가하는 방법

```ts
//패키지 설치
npm install web3  // npm으로 설치

yarn add web3 // yarn으로 설치

bower install web3 // bower로 설치
```

```js
// 깃허브에서 다운로드해 포함
<script language="javascript" type="text/javascript" src="web3.min.js"></script>
```

---

## Web3 프로바이더(Provider)

> 이더리움은 똑같은 데이터의 복사본을 공유하는 *노드*들로 구성되어 있음, 프로바이더를 설정하는 것은 우리 코드에 읽기/쓰기를 처리하려면 어떤 노드와 통신해야 하는지 설정하는 것

### Infura

> 빠른 읽기를 위한 캐시 계층을 포함하는 다수의 이더리움 노드를 운영하는 서비스, 무료 API 사용 가능

```js
var web3 = new Web3(
  new Web3.providers.WebsocketProvider('wss://mainnet.infura.io/ws')
); //다음과 같이 프로바이더 Infura를 쓰도록 설정
```

많은 사용자들이 DApp에서 읽기 연산만 쓰는게 아닌 쓰기 연산도 씀 => 사용자들이 개인 키로 트랜잭션에 서멍할 수 있도록 해야함

> 참고 : 이더리움(블록체인)은 트랜잭션에 전자 서명을 하기 위해 공개/개인 키 쌍 사용

### Metamask

> 사용자들이 이더리움 계정과 개인 키를 안전하게 관리할 수 있게 해주는 크롬, 파이어폭스의 확장 프로그램, 해당 계정들을 써서 Web3.js를 사용하는 웹사이트들과 상호작용할 수 있도록 해줌

> 참고 : 메타마스크는 내부적으로 Infura의 서버를 Web3 프로바이더로 사용 (+ 사용자에게 프로바이더를 선택하게 옵션제공)

```js
window.addEventListener('load', function () {
  // Web3가 브라우저에 주입됬는지 확인(Mist/Metamask)
  if (typeof web3 !== 'undefined') {
    // Mist/Metamask의 프로바이더 사용
    web3.js = new Web3(web3.currentProvider);
  } else {
    // 사용자가 Metamask를 설치하지 않은 경우 처리
    // 사용자들에게 Metamask를 설치하라는 등의 메시지를 보여줄 것
  }

  startApp();
});
```

web3라는 전역 자바스크립틑 객체를 통해 브라우저에 Web3 프로바이더를 주입, web3가 존재하는지 확인 후 적용

> Mist 웹 브라우저 : 메타마스크 외에 사용자들이 쓸 수 있는 개인 키 관리 프로그램

---

## 컨트랙트와 대화

> **컨트랙트의 주소, ABI** : Web3.js가 스마트 컨트랙트와 통신하기 위해 필요한 2가지

1. 컨트랙트 주소 : 컨트랙트 배포 이후 가지는 이더리움 상에 고정된 주소

2. 컨트랙트 ABI : (Application Binary Interface) JSON 형태로 컨트랙트의 메소드를 표현, 컨트랙트 배포하기 위해 컴파일 시 컴파일러가 줌

```js
//myContract(Web3.js) 컨트랙트 인스턴스화
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```

---

## 컨트랙트 함수 호출하기

> Web3.js는 컨트랙트의 함수를 호출하기 위해 2개의 메소드를 사용 : call / send

1. Call : view, pure 함수를 위해 사용, 로컬 노드에서만 실행, 블록체인 트랜잭션을 만들지 않음

```js
// 123 매개변수로 myMethod 함수를 call
myContract.methods.myMethod(123).call();
```

2. Send : 트랜잭션을 만들고 블록체인 상의 데이터를 변경, view와 pure를 제외한 모든 함수에 사용

> 참고 : 트랜잭션을 send 하는 것은 사용자에게 가스를 지불하도록 함, (메타마스크에서 서명하라고 뜸)

```js
// 123 매개변수로 myMethod 함수를 호출하는 트랜잭션 send
myContract.methods.myMethod(123).send();
```

---

### 컨트랙트의 데이터 받기

> 컨트랙트 내 public 으로 선언된 변수는 자동으로 같은 이름의 퍼블릭 "getter" 함수를 만들어 냄

```js
function getZombieDetails(id) {
  return cryptoZombies.methods.zombies(id).call();
}

getZombieDetails(15).then(function (result) {
  console.log('Zombie 15: ' + JSON.stringfy(result));
});
```

위의 실행 결과는 Web3 프로바이더와 통신하여 데이터를 가져옴, 즉 이는 외부 서버로 API 호출을 하는 것처럼 **비동기적**으로 일어남

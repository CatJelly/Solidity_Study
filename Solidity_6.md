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

위의 실행 결과는 Web3 프로바이더와 통신하여 데이터를 가져옴, 즉 이는 외부 서버로 API 호출을 하는 것처럼 **비동기적**으로 일어남, Web3는 여기서 Promise를 반환

```js
// 만들어진 Promise에 then문장 실행 후 result
{
  "name": "H4XF13LD MORRIS'S COOLER OLDER BROTHER",
  "dna": "1337133713371337",
  "level": "9999",
  "readyTime": "1522498671",
  "winCount": "999999999",
  "lossCount": "0" // Obviously.
}
```

---

## 메타마스크 & 계정

> 메타마스크에서 사용자 계정 가져오기

메타마스크는 확장 프로그램 안에서 사용자들이 다수의 계정을 관리할 수 있도록 해줌

```js
// 주입돼 있는 web3 변수에 현재 활성화된 계정이 무엇인지 확인 가능
var userAccount = web3.eth.accounts[0];
```

사용자가 언제든 활성화된 계정을 바꿀 수 있기 떄문에 이 변수값이 바뀌는지 계속 감시해야함 바뀌면 그에 따라 업데이트

```js
// 100 밀리초마다 userAccount 감시
var accountInterval = setInterval(function () {
  // 계정이 바뀌었는지 확인
  if (web3.eth.accounts[0] !== userAccount) {
    userAccount = web3.eth.accounts[0];
    // 새 계정에 대한 UI로 업데이트하기 위한 함수 호출
    updateInterface();
  }
}, 100);
```

---

## 데이터 보여주기

> 컨트랙트로부터 받은 데이터를 보여주기 위한 방법

jQuery를 이용해 파싱하고 표현하는 예제 [페이지](https://github.com/loomnetwork/cryptozombies-lesson-code/blob/master/lesson-6/chapter-6/index.html)

---

## 트랜잭션 보내기

> send 함수를 통해 스마트 컨트랙트의 데이터 변경

- 트랜잭션을 전송(send)하려면 함수 호출한 사람의 from 주소가 필요(솔리디티는 msg.sender)
- 트랜잭션 전송(send)은 가스를 소모함
- 사용자가 트랜잭션 전송을 하고 난 후 실제로 적용까지는 지연이 발생

이런 코드의 비동기적 특성을 다루기 위한 로직이 필요

```js
function createRandomZombie(name) {
  // 시간이 꽤 걸릴 수 있으니, 트랜잭션이 보내졌다는 것을
  // 유저가 알 수 있도록 UI를 업데이트해야 함
  $('#txStatus').text(
    'Creating new zombie on the blockchain. This may take a while...'
  );
  // 우리 컨트랙트에 전송하기:
  return CryptoZombies.methods
    .createRandomZombie(name)
    .send({ from: userAccount })
    .on('receipt', function (receipt) {
      $('#txStatus').text('Successfully created ' + name + '!');
      // 블록체인에 트랜잭션이 반영되었으며, UI를 다시 그려야 함
      getZombiesByOwner(userAccount).then(displayZombies);
    })
    .on('error', function (error) {
      // 사용자들에게 트랜잭션이 실패했음을 알려주기 위한 처리
      $('#txStatus').text(error);
    });
}
```

- receipt : 트랜잭션이 이더리움의 블록에 포함될 때(좀비가 생성되고 우리 컨트랙트에 저장될 때) 발생
- error : 트랜잭션이 블럭에 포함되지 못했을 때(사용자가 충분한 가스를 전송하지 않았을 때) 발생

> !참고 : send 호출 시 gas와 gasPrice를 선택적으로 지정할 수 있음 _.send({ from: userAccount, gas: 3000})_ 지정하지 않으면 메타마스크가 값을 선택할 수 있도록 함

---

## Payable 함수 호출

> Wei : 이더의 가장 작은 하위 단위 (1Eth = 10^18 wei)

```js
// 1ETH 를 Wei로 바꿈
web3js.utils.toWei('1');
```

```js
// 0.001 이더를 보내는 예제
CryptoZombies.methods
  .levelUp(zombieId)
  .send({ from: userAcount, value: web3js.utils.toWei('0.001') });
```

---

## Event 구독하기

> Web3.js 에서 이벤트를 구독해 해당 이벤트가 발생할 때마다 Web3 프로바이더가 코드내 특정 로직을 실행시키도록 할 수 있음

```js
cryptoZombies.events
  .NewZombie()
  .on('data', function (event) {
    let zombie = event.returnValues;
    // `event.returnValue` 객체에서 이 이벤트의 세가지 반환 값에 접근할 수 있음
    console.log('새로운 좀비가 태어났습니다!', zombie.zombieId, zombie.name);
  })
  .on('error', console.error);
```

위 코드는 모든 좀비가 생성될 때마다 알림을 보냄

### indexed 사용

> 이벤트를 필터링하고 현재 사용자와 연관된 변경만을 수신하기 위해 ERC721을 구현할 때 Transfer 이벤트에서 했던 것처럼 _indexed_ 키워드를 사용해야 함

```ts
event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
```

\_from, \_to 가 indexed 되어 있어 프론트엔드 이벤트 리스너에서 필터링할 수 있음

```js
// `filter`를 사용해 `_to`가 `userAccount`와 같을 때만 코드를 실행
cryptoZombies.events
  .Transfer({ filter: { _to: userAccount } })
  .on('data', function (event) {
    let data = event.returnValues;
    // 현재 사용자가 방금 좀비를 받음
    // 해당 좀비를 보여줄 수 있도록 UI를 업데이트할 수 있도록 여기에 추가
  })
  .on('error', console.error);
```

### 지난 이베늩에 대해 질의하기

- getPastEvents : 지난 이벤트들에 대해 질의
- fromBlock, toBlock : 저런 필터를 이용해 이벤트 로그에 대한 시간 범위를 전달 가능

```js
cryptoZombies
  .getPastEvents('NewZombie', { fromBlock: 0, toBlock: 'latest' })
  .then(function (events) {
    // `events`는 우리가 위에서 했던 것과 같은 반복 접근할 `event` 객체들의 배열
    // 이 코드는 생성된 모든 좀비의 목록을 우리가 받을 수 있게 할 것임
  });
```

> 위 메소드를 활용해서 이벤트 로그들에 대한 질의 가능 => 이벤트를 저렴한 형태의 storage로 사용

- 스마트 컨트랙트 안에서는 이벤트를 읽을 수 없음
- 히스토리로 블록체인에 기록하여 읽기 원하는 데이터가 있다면 기록용으로 사용 가능

---

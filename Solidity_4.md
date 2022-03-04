# Chapter 4 (Solidity Advanced)

## 함수 제어자 Payable

> 함수 제어자 정리 :

1. 접근 제어자(visibility modifier) :
   - private : 컨트랙트 내부의 다른 함수들에 의해서만 호출
   - internal : private + 상속한 컨트랙트에서도 호출 가능
   - public : 내외부 모두, 어디서든 호출 가능
2. 상태 제어자(state modifier) : 모두 컨트랙트 외부에서 불렸을 때 가스 소모하지 않음 _(! 다른 함수에 의해 내부적 호출인 경우 가스 소모)_
   - view : 해당 함수를 실행해도 어떤 데이터도 저장/변경되지 않음
   - pure : 해당 함수가 어떤 데이터도 블록체인에 저장하지 않음, 블록체인으로부터 어떤 데이터도 읽지 않음
3. 사용자 정의 제어자 : ex) onlyOwner, aboveLevel

```ts
function test() external view onlyOwner anotherModifier { /* ... */ }
```

---

> payable 제어자 : 이더를 받을 수 있는 특별한 함수 유형

일반적인 웹 서버에서 API 함수를 실행 시에는 함수 호출을 통해 US 달러를 보낼수 없음(비트코인 포함) <br>
**하지만** 이더리움은 돈(_이더_), 데이터(transaction payload), 컨트랙트 코드 자체 모두 이덜리움에 존재하기 때문에 _함수 실행하는 동시에 컨트랙트에 돈을 지불하는 것이 가능_

```ts
contract OnlineStore {
    function buySomething() external payable {
        // 함수 실행에 0.001 이더가 보내졌는지 확실히 하기 위해 확인
        require(msg.value == 0.001 ether);
        // 보내졌다면 함수 호출한 자에게 디지털 아이템을 전달하기 위한 내용 구성;
        transferThing(msg.sender);
    }
}
```

- msg.value : 컨트랙트로 이더가 얼마나 보내졌는지 확인하는 방법 (ether : 기본적으로 포함된 단위)

- 여기서 발생한 일은 누군가 web3.js에서 다음과 같이 함수 실행 시 발생

```ts
// 'OnlineStore' 는 이더리움 상의 컨트랙트를 가리킨다고 가정
OnlineStore.buySomething({
  from: web3.eth.defaultAccount,
  value: web3.utils.toWei(0.001),
});
```

- value 필드 : ether를 얼마나 보낼지 결정
- 트랜잭션을 봍우로 생각, 함수 호출에 전달하는 매개변수를 편지 내용이라 할 때 value는 봉투안에 현금을 넣는 것

**!참고 : payable 함수가 아닌데 value를 포함할 시 트랜잭션 거부**

---

## 출금

> 컨트랙트로 이더를 보내면 해당 컨트랙트의 이더리움 계좌에 이더가 저장 => 이더를 인출하는 함수를 만들어야 함

```ts
contract GetPaid is Ownable {
    // 컨트랙트에서 이더를 인출하는 함수
    function withDraw() external onlyOwner {
        owner.transfer(this.balance);
    }
}
```

- this.balance : 컨트랙트에 저장돼있는 전체 잔액
- transfer 함수 : 이더를 특정 주소로 전달할 수 있음

```ts
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```

어떤 아이템에 대해 초과 지불 했다면 msg.sender로 되돌려줄 수도 있음

```ts
seller.transfer(msg.value);
```

구매자, 판매자가 존재하는 컨트랙트에서 판매자 주소를 storage에 저장하고 누군가 판매자 아이템을 구매하면 요금을 판매자에게 전달 가능 (분산 장터)

---

## 난수(Random Numbers)

> 솔리디티에서 난수 발생하는 방법은 안전하게 할 수 없음

### keccak256을 통한 난수 생성 : 가장 좋은 방법

```ts
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```

- 위 예시는 now, msg.sender, randNonce(딱 한번 쓰이는 숫자)를 매개변수로 씀
- Keccak256을 사용해 입력들을 임의의 해시값으로 변환, 변환된 값을 uint로 바꾸고 % 100을 써 마지막 2자리 숫자만 취함 (0~99 난수)

> 이 메소드는 정직하지 않은 노드의 공격에 취약

1. 이더리움에서는 컨트랙트의 함수를 실행하면 **트랜잭션(transaction)** 으로서 네트워크의 노드들에게 실행을 알림
2. 그 후 네트워크의 노드들은 여러개의 트랜잭션을 모으고 "작업 증명"이라 하는 매우 복잡한 수학적 문제를 풀기 위한 시도를 함
3. 해당 트랜잭션 그룹을 그들의 작업증명(PoW) + 블록으로 네트워크에 배포
4. 한 노드가 어떤 PoW를 풀면 다른 노드들은 그 PoW를 풀려는 시도를 멈추고 해당 노드가 보낸 트랜잭션 목록이 유효한지 검증
5. 유효하면 해당 블록을 받아들이고 다음 블록을 풂

> 이것이 난수 함수를 취약하게 만듬

나의 노드에서만 임의의 작업을 하고 원하는 바를 이룰 때까지 트랜잭션을 공유하지 않고 나의 노드에서만 무한정 시도할 경우가 있음

---

### 안전하게 난수를 만들어내는 방법

> 나중에 [사이트](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract) 참고할 것 <br>
> (외부의 난수 함수에 접근할 수 있도록 오라클을 사용하는 것)

---

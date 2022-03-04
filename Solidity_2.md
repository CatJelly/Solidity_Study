# Chapter 2 (Solidity Basic)

## 주소와 매핑

> **주소** : 이더리움은 은행 계좌같은 계정들로 이뤄져 있음 계정은 이더리움 블록체인의 통화인 *이더*의 잔액을 가짐, 각 계정은 **주소**를 가짐

> **매핑** : 구조화된 데이터를 저장하는 방법

```ts
// 금융 앱용, 유저의 계좌 잔액을 보유하는 uint를 저장함
mapping (address => uint) public accountBalance;
// userId로 유저일므을 저장/검색하는데 매핑을 쓸 수 있음
mapping (Uint => string) userIdToName;
```

- mapping : 키-값 저장소, 데이터 저장, 검색하는데 이용

## Msg.sender

> 현재 함수를 호출한 사람의 주소를 가리킴 (모든 함수에서 이용 가능한 특정 전역 변수중 하나)

! 참고 :
솔리디티에서 함수 실행은 항상 외부 호출자가 시작

```ts
mapping (address => uint) favoriteNUmber;

function setMyNumber(Uint _myNumber) public {
    favoriteNumber[msg.sender] = _myNumber;
}

function whatIsMyNumber() public view return (uint) {
    return favoriteNumber[msg.sender];
}
```

---

## Require

> 특정 조건이 참이 아닐 때 함수가 에러 메시지를 발생 후 실행 멈춤

```ts
function sayHiToVitalik(string _name) public {
    require(keccak256(_name) == keccak256("Vitalik"));
    return "Hi!";
}
```

- 함수 실행 전에 참인지 확인하는데 유용

---

## 상속

> 하나의 큰 컨트랙트를 만들기 보단 여러 컨트랙트에 로직을 나누는 것이 합리적

```ts
contract Doge {
    function catchphrase() public returns (string) {
        return "So Wow CryptoDoge";
    }
}

...

import "./Doge.sol"; //Doge.sol 불러오기

contract BabyDoge is Doge {
    function anotherCatchphrase() public returns (string) {
        return "Such Moon BabyDoge";
    }
}
```

---

## Storage vs Memory

> 변수를 젖아할 수 있는 공간의 두 가지

- storage : 블록체인 상에 영구적으로 저장되는 변수
- memory : 임시적으로 저장되는 변수, 컨트랙트 함수에 대한 외부 호출이 일어나는 사이 지워짐

```ts
contract SandwichFactory {
    struct Sandwich {
        string name;
        string status;
    }
    Sandwich[] sandwiches;

    function eatSandwich(uint _index) public {
        // Sandwich mySandwich = sandwiches[_index];
        // `storage`나 `memory`를 명시적으로 선언해야 한다는 경고 메시지를 발생한다.
        Sandwich storage mySandwich = sandwiches[_index];
        // ...이 경우, `mySandwich`는 저장된 `sandwiches[_index]`를 가리키는 포인터이다.
         mySandwich.status = "Eaten!";
        // ...이 코드는 블록체인 상에서 `sandwiches[_index]`을 영구적으로 변경한다.

        // 단순히 복사를 하고자 한다면 `memory`를 이용하면 된다:
        Sandwich memory anotherSandwich = sandwiches[_index + 1];
        // ...이 경우, `anotherSandwich`는 단순히 메모리에 데이터를 복사하는 것이 된다.
        anotherSandwich.status = "Eaten!";
        // ...이 코드는 임시 변수인 `anotherSandwich`를 변경하는 것으로
        // `sandwiches[_index + 1]`에는 아무런 영향을 끼치지 않는다.
        sandwiches[_index + 1] = anotherSandwich;
        // ...이는 임시 변경한 내용을 블록체인 저장소에 저장하고자 하는 경우이다.
    }
}
```

---

## 함수 접근 제어자

> private, public 외 접근 제어자가 있음(상속)

- **internal** : 함수가 정의된 컨트랙트를 상속하는 컨트랙트에서도 접근 가능 + private
- **external** : 함수가 컨트랙트 바깥에서만 호출 될 수 있고 컨트랙트 내 다른 함수에 의해 호출될 수 없음

---

## 다른 컨트랙트와 상호작용

> 소유하지 않은 컨트랙트와 우리 컨트랙트가 상호작용 하기 위해 **인터페이스** 정의 필요

```ts
contract LuckyNumber {
    mapping(address => uint) numbers;

    function setNum(uint _num) public {
        numbers[msg.sender] = _num;
    }

    function getNum(address _myAddress) public {
        return numbers[_myAddress];
    }
}
```

아무나 자신의 행운의 수를 저장할 수 있는 간단한 컨트랙트

```ts
contract NumberInterface {
    function getNum(address _myAddress) public view returns (uint);
}
```

컨트랙트와 상호작용하고자 하는 함수만 선언한 형태인 **인터페이스**

---

## 다수의 반환값 처리

> 함수가 여러개 값을 반환할 때 특정 값만 취하는 방법

```ts
function multipleReturns() internal returns(uint) {
    return (1,2,3);
}

function processMultipleReturns() external {
    uint a;
    uint b;
    uint c;
    (a,b,c) = multipleReturns(); // 다수값 할당
}

function getLastReturnValue() external {
    uint c;
    (,,c) = multipleReturns();
}
```

---

## if 문

> 자바스크립트의 if문과 동일

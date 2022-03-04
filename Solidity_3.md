# Chapter 3 (Solidity Advanced)

## 소유 가능한 컨트랙트

> 컨트랙트를 소유 가능하게 만듬, 컨트랙트를 대상으로 특별한 권리를 가지는 소유자가 있음을 의미

> OpenZeppelin - Ownable 컨트랙트 : 솔리디티 라이브러리 OpenZeppelin 의 Ownable 컨트랙트, 대부분의 DApp들은 Ownable 컨트랙트를 복붙하며 시작

- modifier(함수 제어자) : 다른 함수들에 대한 접근을 제어하기 위해 사용되는 유사 함수 (함수 실행 전의 요구사항 충족 여부 확인)
  function 대신 modifier씀, 함수처럼 직접 호출 X,

```ts
modifier onlyOwner() {
    require(msg.sernder == owner);
    _;
}
```

```ts
contract MyContract is Ownable {
    event LaughManiacally(string laughter);

    function likeABoss() external onlyOwner {
        LaughManiacally("Muahahahahaha");
    }
}
```

- likeABoss 함수의 onlyOwner 제어자 활용 예
- 오직 컨트랙트의 소유자만 해당 함수를 호출할 수 있음

---

## 가스(Gas)

> 이더리움 DApp이 사용하는 연료, DApp의 함수를 실행 시마다 **가스**라 불리는 화폐를 지불, 사용자는 이더(ETH, 이더리움 화폐)를 이용해 가스를 구입

- 각 연산에 따라 소모되는 가스비용(gas cost)이 다름

* 코드 최적화가 매우 중요

> 가스가 필요한 이유 : 이더리움 네트워크 상의 모든 개별 노드가 함수 출력값 검증을 위해 실행해야함 => 이 과정에서 연산처리에 비용이 들도록 해야 무차별적 공격 방지 가능

```ts
struct NoramlStruct {
    uint a;
    uint b;
}

struct MiniMe {
    uint32 a;
    uint32 b;
}
```

- 이런식으로 구조체 압축 가능

---

## 시간 단위

> 솔리디티는 시간을 다룰 수 있는 단위계를 제공함

- now : 현재 유닉스 타임스탬프값 (1970년 1월 1일부터 지금까지 초단위 합)
- seocnds, minutes, hours, days, weeks, years (uint)

```ts
uint lastUpdated;

function updateTimestamp() public {
    lastUpdated = now;
}

//updateTimestamp가 불린지 5분이 넘었으면 true 아니면 false 리턴
function fiveMinutesHavePassed() public view returns (bool) {
    return (now >= (lastUpdated + 5 minutes));
}

```

---

## 구조체 인수로 전달

> private, internal 함수에 인수로 구조체의 storage 포인터를 전달 가능, (함수들 간에 특정 구조체를 주고 받을 때 유용)

```ts
function _doStuff(Zombie storage _zombie) internal {
    // _zombie로 할 수 있는 것 처리
}
```

---

## Public 함수 & 보안

> 코드의 보안을 점검하는 방법 :

1. public, external 함수를 검사
2. 사용자들이 해당 함수를 남용할 수 있는 방법 생각
3. onlyOwner같은 제어자를 안가지면 원하는 모든 데이터를 취할 수 있는지

---

## 함수 제어자의 또 다른 특징

> 인수를 가지는 함수 제어자 :

```ts
// 사용자의 나이를 저장하기 위한 매핑
mapping (uint => uint) public age;

// 사용자가 특정 나이 이상인지 확인하는 제어자
modifier olderThan(uint _age, uint _userId) {
    require(age[_userId] >= _age);
    _;
}

//차를 운전하기 위해서는 16살 이상이여야 함
function drivecar(uint _userId) public olderThan(16, _userId) {
    // 필요한 함수 내용
}
```

---

## View 함수를 사용해 가스를 절약

> 블록체인에서 데이터를 읽기만 하면 되는 경우 가스 소모를 절감하기 위해 사용,
> View 함수는 가스를 소모하지 않음
>
> > = web3.js에 "이 함수는 실행할 때 로컬 이더리움 노드에 질의만 날리면 됨 && 블록체인에 어떤 트랜잭션도 만들지 않음"

**!참고** : 만약 view 함수가 동일 컨트랙트 내에 있는 view 함수가 아닌, 다른 함수에서 내부적으로 호출될 경우, 여전히 가스 소모! (외부에서 호출 됐을 때에만 무료)

---

## Storage는 비싸다

> 솔리디티에서 비싼 연산 = storage를 쓰는것 => 그중에서도 쓰기 연산

데이터의 일부를 쓰거나 바꿀 떄마다 블록체인에 영구적으로 기록되기 때문..  
=> 비용을 최소화하기 위해 storage에 데이터를 가급적이면 안쓸 것  
_ex) 어떤 배열에서 내용을 빠르게 찾기 위해 단순히 변수에 저장하는 것 대신 함수가 호출될 때마다 배열을 memory에 다시 만듬_

> 대부분의 프로그래밍 언어는 큰 데이터 집합의 개별 데이터에 모두 접근하는 방식은 비용이 비쌈,  
> **솔리디티에서는 그 접근이 _external view_ 함수라면 *storage*를 사용하는것 보다 저렴**

### 메모리에 배열 선언하기

```ts
function getArray() external pure returns(uint[]) {
    // 메모리에 길이 3의 새로운 배열 생성
    uint[] memory values = new uint[](3);

    // 특정한 값들 넣기
    values.push(1);
    values.push(2);
    values.push(3);

    // 해당 배열을 반환
    reutrn values;
}
```

> 메모리 배열은 storage 배열 처럼 array.push()로 크기가 조절되지 않음

---

## For 반복문

```ts
mapping (address => uint[]) public ownerToZombies
```

```ts
function getZombiesByOwner(address _owner) external view returns (uint[]) {
    return ownerToZombies[_owner];
}
```

> 위 방식의 문제 : 한 좀비를 원래 소유자에서 다른 사람에게 전달해야하는 함수를 구현한다면

1. 전달할 좀비를 새로운 소유자의 ownerToZombies 배열에 넣음
2. 기존 소유자의 ownerToZombies 배열에서 해당 좀비를 지움
3. 지워진 구간을 메우기 위해 한 칸씩 움직임, 배열 크기 1감소

> view 함수는 외부 호출시 가스 소모 없음, 따라서 **for 반복문**을 사용해 특정 사용자 좀비로 구성된 배열을 만들어서 리턴

```ts
function getZombiesByOwner(address _owner) external view returns(uint[]) {
    uint[] memory result = new uint[](ownerZombieCount[_owner]);
    uint counter = 0;
    for(uint i=0; i<zombies.length; i++) {
        if(zombieToOwner[i] == _owner) {
            result[counter] = i;
            counter++;
        }
    }
    return result;
}
```

---

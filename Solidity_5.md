# Chapter 5 (Solidity Advanced)

## 이더리움 상의 토큰

> 토큰(_ERC20 토큰_) : 공통 규약을 따르는 스마트 컨트랙트, (즉 모든 다른 토큰 컨트랙트가 사용하는 표준 함수 집합을 구현한 것)

- ERC20 토큰은 화폐처럼 사용되는 토큰으로 매우 적절
- ERC721 토큰은 각각이 유일하고 분할 불가능(교체 불가능), 각각은 유일한 ID를 가짐

```ts
//ex)
transfer(address _to, uint256 _value)
balance(address _owner)
```

즉 토큰은 하나의 컨트랙트, 그 안에 누가 얼마나 만흥 토큰을 가지고 있는지 기록, 몇몇 함수를 가지고 사용자들이 그들의 토큰을 다른 주소로 전송할 수 있게 해줌

---

### 이렇게 하는 이유

> 모든 ERC20 토큰들이 같은 이름의 동일한 함수 집합을 공유하기 때문에 동일한 방식으로 상호작용 가능

즉 하나의 ERC20 토큰과 상호작용할 수 있는 앱을 하나 만들면 이 앱은 다른 어떤 ERC20 토큰과도 상호작용이 가능

---

## ERC721 표준

```ts
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```

> 토큰 컨트랙트를 구현 :

1. 인터페이스를 솔리디티 파일로 따로 복사해 저장
2. import "./erc721.sol"; 로 임포트
3. 해당 컨트랙트를 상속하는 우리 컨트랙트를 만듬
4. 각각의 함수를 오버라이딩

---

## 다중 상속

```ts
contract SatoshiNakamoto is NickSzabo, HalFinney {
   // 다중 상속 사용 시 상속하고자 하는 다수 컨트랙트에 ,로 구분
}

```

---

## balanceOf & ownerOf

> balanceOf

```ts
function balanceOf(address _owner) public view returns (uint256);
```

: address 를 받아, 해당 주소가 토큰을 얼마나 가지고 있는지 반환

> ownerOf

```ts
function ownerOf(uint256 _tokenId) public view returns (address);
```

: 토큰 ID를 받아 이를 소유하고 있는 사람의 주소를 반환

---

## ERC721 : 전송 로직

> 토큰을 전송하는 2가지 방법

```ts
function transfer(address _to, uint256 _tokenId) public;
function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```

1. 토큰의 소유자가 전송상대의 address, 전송하고자 하는 tokenId와 함께 transfer 함수를 호출
2. 토큰의 소유자가 먼저 위 정보를 가지고 approve 호출하고 컨트랙트에서 누가 해당 토큰을 가질 수 있도록 허가를 받았는지 저장 (보통 mapping (uint256 => address) 사용) 이후 누군가 takeOwnership 호출하면 해당 컨트랙트는 msg.sender가 소유자로부터 토큰을 받을 수 있게 허가받았는지 확인 후 토큰 전송 결정

1, 2 둘다 동일한 로직인데 순서만 반대 _(1은 토큰 전송자가 함수 호출, 2는 토큰 받는 사람이 호출)_

---

## ERC721 : Transfer

> \_transfer(private)를 만들어 추상화 한 뒤 transfer(pulbic)을 구현

---

## ERC721 : Approve & TakeOwnership

> approve / takeOwnership 사용은 2단계

1. 소유자가 새로운 소유자의 address와 보내고 싶은 tokenId를 사용해 approve 호출
2. 새로운 소유자가 tokenId를 사용해 takeOwnership 함수를 호출하면 컨트랙트는 승인된 자인지 확인 후 전송

!참고 : 2번의 함수 호출로, 함수 호출 사이 누가 무엇에 대해 승인이 되었는지 저장할 데이터 구조 필요

---

## 오버플로우 방지

> 컨트랙트 보안 강화 : 오버플로우 / 언더플로우

- 오버플로우 : 저장 공간의 크기를 벗어나는 값을 대입 했을 때 발생
- 언더플로우 : 오버플로우의 반대 경우

> SafeMath : OpenZeppelin에서 이런 상황을 방지하기 위해 만든 라이브러리

```ts
import "./safemath.sol";
...

using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```

- SafeMath 라이브러리를 사용하는 방법
- SafeMath 는 4개의 함수를 가짐 - add, sub, mul, div

> library : contract와 비슷하지만 using 키워드를 사용할 수 있게 해줌

> assert : require과 비슷하지만, [ require의 경우 함수 실행 실패하면 남은 가슬르 사용자에게 반환 /assert는 그렇지 않음

---

## 주석(Comment)

> 자바스크립트와 비슷한 방법

```
// 한 줄 주석, 자신, 다른 사람에 대한 메모와 같음
/*
여러줄 주석하는 방법
여러줄 주석하는 방법
*/
```

> natspec : 솔리디티 커뮤니티 표준 형식 주석 설명

```ts
/// @title 기본적인 산수를 위한 컨트랙트
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice 지금은 곱하기 함수만 추가한다.
contract Math {
  /// @notice 2개의 숫자를 곱한다.
  /// @param x 첫 번쨰 uint.
  /// @param y 두 번째 uint.
  /// @return z (x * y) 곱의 값
  /// @dev 이 함수는 현재 오버플로우를 확인하지 않는다.
  function multiply(uint x, uint y) returns (uint z) {
    // 이것은 일반적인 주석으로, natspec에 포함되지 않는다.
    z = x * y;
  }
}
```

- @notice : 사용자에게 컨트랙트/함수가 무엇을 하는지 설명
- @dev : 개발자에게 추가적인 상세 정보 설명
- @param : 함수의 매개변수
- @return : 함수의 반환값

모든 태그는 항상 필수가 아님, @dev에 각 함수가 어떤 기능을 하는지는 명시해줄 것

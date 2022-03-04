# Chapter 1 (Solidity Basic)

---

```ts
pragma solidity ^0.4.19;

contract HelloWorld {

}
```

- 솔리디티 코드는 **컨트랙트** 안에 싸여 있음

* 모든 솔리디티 소스코드는 **version pragma**로 시작해야 함

---

## 상태 변수 & 정수

> 상태변수 :
> 컨트랙트 저장소에 영구적 저장 (이더리움 블록체인에 기록)

```ts
contract Example {
    uint myUnsignedInteger = 100;
    //
}
```

- uint ( = uint256 ) : 부호 없는 정수, uint8, uint16, uint32 도 가능

---

## 수학 연산, 형변환

> 덧셈, 뺄셈, 곱셈, 나눗셈, 나머지, 지수연산(\*\*) 지원

```ts
uint8 a = 5;
uint b = 6;
uint8 c = a * b; // uint를 반환해서 에러
uint8 c = a * uint8(b); // b를 형변환해 에러 해결

```

---

## 구조체

> 여러 특성을 가진 복잡한 자료형 생성 가능

```ts
struct Person {
    uint age;
    string name; //utf-8 데이터
}
```

---

## 배열

```ts
uint[2] fixedArray; // 정적 배열
uint[] dynamicArray; // 동적 배열
Person[] people; // 구조체 동적 배열
```

> 동적 배열은 고정된 크기가 없고 계속 커질 수 있음

```ts
Person[] public people;
people.push(Person(16, "Person1"));
```

> public으로 배열 선언 가능 (getter 메소드 자동 생성), 다른 컨트랙트들이 이 배열을 읽을 수 있음(쓰기 X), 일종의 공개 데이터용

- array.push() : 배열 끝에 추가해 순서 유지

---

## 함수 선언

> 함수는 기본적으로 public 으로 선언됨

```ts
function eatHamburgers(string _name, uint _amount) {

}
```

- 인자명은 \_로 시작해 전역변수와 구별

> private 함수는 컨트랙트 내 다른 함수들만이 호출 가능

```ts
uint[] numbers;

function _addTOArray(uint _number) private {
    numbers.push(_number);
}
```

- private 함수명은 \_로 시작함

```ts
string greeting = "What's up dog";

function sayHello() public returns (string) {
    return greeting;
}

function sayHello() public view returns (stirng) {
    return greeting;
}

function _multiply(Uint a, uint b) private pure returns (uint) {
    return a * b;
}

```

- 함수 선언에서 반환값 종류를 포함함 (위의 경우 string)
- view : 상태를 변화시키지 않는 함수(값 변경, 쓰기 X)
- pure : 함수가 앱에서 어떤 데이터도 접근하지 않음

---

## Keccak256

> keccak256 : SHA3의 한 버전인 내장 해시 함수, 입력 스트링을 랜덤 256비트 16진수로 매핑

---

## 이벤트

> 컨트랙트가 블록체인 상에서 앱의 사용자 단에서 뭔가 액션이 발생했을 때 의사소통 하는 방법

```ts
event IntegersAdded(uint x, uint y, uint result); // 이벤트 선언

function add(Uint _x, uint _y) public {
    uint result = _x + _y;
    // 이벤트를 실행해 앱에게 add 함수가 실행됬음을 알림
    IntegersAdded(_x, _y, result);
    return result;
}
```

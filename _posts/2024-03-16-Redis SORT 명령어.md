---
layout: post
title: "[Redis] SORT 명령어"
date: 2024-03-15 15:00:00
categories: Redis
---

---

## 관계형 데이터 로드 전략

Redis에서 관계형 데이터를 로드하는 전략은 2가지가 있다.

1. 반환된 모든 ID들마다 HGETALL을 한번씩 실행하는 단일 Pipeline을 생성하는 방법
	- 즉, 파이프라인으로 다수의 명령어를 실행한 후 하나의 배치로 Redis에 전송해서 하나의 큰 배치로 응답받는 방법이다.
	- 단점: 데이터를 로드하기 위해 2개의 요청을 별도로 호출해야 한다. (key를 가져온 후, 해당 key들로 HGETALL을 실행)
2. `SORT` 명령어를 사용하는 방법
	- 매우 효율적인 솔루션으로, 지금부터 알아보겠다.

---

## 관계형 데이터에 SORT 명령어 사용하기

`SORT` 명령어를 사용하여 여러 소스의 데이터를 하나의 구조로 결합할 수 있다.

---

## SORT 명령어 간략 소개

- Redis에서 가장 이해하기 까다로운 명령어
- 3가지 데이터 타입에서 사용된다. (Set, Sorted Set, List)
- 이름은 SORT이지만, 정렬 목적이 아닌 다른 목적으로 주로 사용한다.
- Sorted Set에 쓰는 용어들과 상충한다. SORT 명령어에서도 member-score가 쌍을 이룬다. 그러나 SORT 명령어 체계에서 score의 의미는 Sorted Set에서와 다른 의미로 쓰인다.
- SORT 명령어 체계는 member를 score로 취급한다. 즉, member가 정렬 기준이 된다. 예를 들어, Sorted Set을 대상으로 SORT 명령어를 사용할 때, SORT는 Sorted Set의 members에 대해 정렬한다. score 기준으로 정렬하지 않는다.
- 기본적으로 SORT는 숫자를 대상으로 동작한다.

---

## Syntax

```bash
SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern
  ...]] [ASC | DESC] [ALPHA] [STORE destination]
```

- 시간 복잡도
	- `O(N+M*log(M))` (정렬할 때. N: list 또는 set의 요소 개수, M: 반환되는 요소 개수)
	- `O(N)` (정렬하지 않을 때)

Set, Sorted Set, List에 포함된 요소들을 반환하거나 저장한다.

```bash
SORT mylist
```
`mylist`가 숫자 리스트라고 가정했을 때, 이 명령어는 오름차순으로 정렬된 요소들의 리스트를 반환한다.

```bash
SORT mylist DESC
```

내림차순으로 정렬하기 위해서는 `DESC`를 옵션을 사용한다.

```bash
SORT mylist ALPHA
```

`mylist`가 문자열 값을 포함하고 사전식으로 정렬하고 싶은 경우, `ALPHA` 옵션을 사용한다.

Redis는 `UTF-8`을 지원하며, `LC_COLLATE` 환경 변수를 올바르게 설정했다고 가정한다.


`LIMIT` 옵션을 사용하여 반환되는 요소의 수를 제한할 수 있다. 첫 번째로 offset 인수(건너뛸 요소의 수)와 두 번째로 count 인수(offset에서 시작하여 반환할 요소의 수)를 사용한다. 

다음은 mylist의 정렬된 버전에서 요소 0부터 시작하여 10개의 요소를 반환한다.

```bash
SORT mylist LIMIT 0 10
```

---

## SORT 명령어 BY 인수 지정하기

정렬 기준과 조회하려는 값의 종류가 다른 경우

```bash
SORT books:likes BY books:*->year
```

- `books:likes`에는 각 books의 id가 멤버로 저장되어 있고, 각 book은 `books:{id}` 키의 해시맵을 갖는다고 가정한다.
- `*`에는 members가 맵핑되어서, 각 member의 year로 정렬 후 member값을 응답한다.

---

## SORT로 데이터 결합하기

```bash
SORT books:likes BY books:*->year GET books:*->title
```

- `GET books:*->title`: 각 member의 title값 조회
- `GET #`: 원래의 멤버를 조회
- `BY nosort`: 정렬을 안해도 되는 경우. Sorted Set 내의 멤버 순서를 확인해서 데이터 결합 연산 수행

---

## 결론

HGETALL이 훨씬 이해하기 쉽지만, SORT 명령어를 활용하여 Redis에 하나의 요청만 실행해서 원하는 결과를 얻을 수 있어서 더 효율적이다.
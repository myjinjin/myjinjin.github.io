---
layout: post
title: "[Redis] 정렬 집합(Sorted Set) 자료구조"
date: 2024-03-12 15:00:00
categories: Redis
---

---

Redis 정렬 집합(Sorted Set) 자료구조는 score에 따라 정렬된 unique한 문자열(member)의 컬렉션 자료구조이다. 동일한 score를 가진 문자열이 여러 개 있는 경우 문자열은 사전식으로 정렬된다.

정렬 집합의 사용 사례는 다음과 같다.

- leaderboard (대규모 온라인 게임에서 가장 높은 점수의 정렬된 목록을 쉽게 유지할 수 있다)
- rate limiter (슬라이딩 윈도우 비율 제한기를 구축하기 위해 정렬 집합을 사용하여 과도한 API 요청을 방지할 수 있다.)

정렬 집합은 집합(Set)과 해시(Hash)의 혼합체로 생각할 수 있다. 집합과 마찬가지로 정렬 집합은 고유하고 반복되지 않는 문자열 요소로 구성되므로 정렬된 집합은 일종의 집합이다.

집합 내의 요소는 정렬되어 있지 않지만, 정렬된 집합의 모든 요소는 `score`라고 불리는 부동 소수점 값과 관련되어 있다. 이 점에서 각 요소가 값을 매핑하는 해시와 유사하다고 할 수 있다.

정렬 집합의 요소는 정렬되어 있는 순서대로 가져온다. 다음 규칙에 따라 정렬된다.

- A.score가 B.score보다 큰 경우 A > B이다.
- B와 A가 동일한 score를 가진 경우 A.string이 B.string보다 사전식으로 더 큰 경우 A > B이다. B와 A 문자열은 정렬 집합에는 고유한 요소만 포함되기 때문에 동일할 수 없다.

---

## 특징

- 모든 멤버는 스코어를 갖고, 스코어에 따라 정렬된다. 스코어는 숫자이다. 
- 가장 유연하고 유용한 데이터 유형 중 하나이다.
- 정렬 집합으로 할 수 있는 일이 매우 많다. 따라서, 명령어도 아주 다양하다.

---

## Example

```bash
> ZADD racer_scores 10 "Norem" # "Norem"이라는 레이서의 score를 10으로 설정하여 racer_scores 정렬 집합에 추가 (1개 추가 성공)
(integer) 1
> ZADD racer_scores 12 "Castilla" # "Castilla"라는 레이서의 score를 12로 설정하여 racer_scores 정렬 집합에 추가 (1개 추가 성공)
(integer) 1
> ZADD racer_scores 8 "Sam-Bodden" 10 "Royce" 6 "Ford" 14 "Prickett" # "Sam-Bodden"의 score를 8로, "Royce"의 score를 10으로, "Ford"의 score를 6으로, "Prickett"의 score를 14로 설정하여 racer_scores 정렬 집합에 각각 추가 (4개 추가 성공)
(integer) 4
```

`ZADD`는 `SADD`와 유사하지만 score라는 추가적인 인수가 있다. 

구현 참고: 정렬 집합은 skip list와 hash table을 모두 포함하는 이중 포트 데이터 구조를 통해 구현된다. 따라서 요소를 추가할 때마다 Redis는 `O(log(N))` 작업을 수행한다. 그러나 정렬된 요소를 요청할 때 Redis는 어떤 작업도 수행할 필요가 없다. 이미 정렬되어 있기 때문이다. 정렬 순서는 `ZRANGE` 낮은 것부터 높은 것, `ZREVRANGE`는 높은 것부터 낮은 것의 순서이다.

```bash
> ZRANGE racer_scores 0 -1 # racer_scores 정렬 집합에서 score 오름차순으로 요소를 가져온다. 시작 인덱스 0, 끝 인덱스 -1로 설정하여서 모든 요소를 가져온다.
1) "Ford"
2) "Sam-Bodden"
3) "Norem"
4) "Royce"
5) "Castilla"
6) "Prickett"
> ZREVRANGE racer_scores 0 -1 # racer_scores 정렬 집합에서 score 내림차순으로 요소를 가져온다. 시작 인덱스 0, 끝 인덱스 -1로 설정하여서 모든 요소를 가져온다.
1) "Prickett"
2) "Castilla"
3) "Royce"
4) "Norem"
5) "Sam-Bodden"
6) "Ford"
```

`WITHSCORES` 인자를 사용하면 score도 함께 반환할 수 있다.

```bash
> ZRANGE racer_scores 0 -1 withscores
 1) "Ford"
 2) "6"
 3) "Sam-Bodden"
 4) "8"
 5) "Norem"
 6) "10"
 7) "Royce"
 8) "10"
 9) "Castilla"
10) "12"
11) "Prickett"
12) "14"
```

범위에 대한 작업을 수행할 수 있다.

```bash
> ZRANGEBYSCORE racer_scores -inf 10 # racer_scores 정렬 집합에서 score가 -inf부터 10까지인 범위의 요소를 가져온다. => score가 10 이하인 모든 레이서를 가져온다.
1) "Ford"
2) "Sam-Bodden"
3) "Norem"
4) "Royce"
```

```bash
> ZREM racer_scores "Castilla" # "Castilla" 레이서를 racer_scores 정렬 집합에서 삭제 (1개 삭제 성공)
(integer) 1
> ZREMRANGEBYSCORE racer_scores -inf 9 # racer_scores 정렬 집합에서 score -inf~9 범위에 있는 요소를 모두 삭제 (2개 삭제 성공)
(integer) 2
> ZRANGE racer_scores 0 -1 # racer_scores 정렬 집합에서 모든 요소 조회
1) "Norem"
2) "Royce"
3) "Prickett"
```

```bash
> ZRANK racer_scores "Norem" # "Norem" 레이서의 racer_scores 정렬 집합에서의 순위 확인 (score 오름차순)
(integer) 0
> ZREVRANK racer_scores "Norem" # "Norem" 레이서의 racer_scores 정렬 집합에서의 역순 순위 확인 (score 내림차순)
(integer) 3
```

---

## score 업데이트하기: 리더보드

정렬 집합의 score는 언제나 업데이트될 수 있다. 정렬 집합에 이미 포함된 요소에 대해 `ZADD`를 호출하면 해당 요소의 점수가 업데이트되며, 요소의 위치도 재조정된다. 이 작업은 `O(log(N))` 시간 복잡도를 갖는다. 따라서 정렬 집합은 요소의 업데이트가 빈번하게 발생하는 경우에도 빠르게 처리할 수 있는 구조이다.

대표적인 예시로, 페이스북 게임을 들 수 있다. 사용자를 점수에 따라 정렬하고 순위 가져오기 작업과 결합하여 상위 N명의 사용자를 표시하고 리더보드에서 사용자의 순위를 보여준다. "당신은 여기서 4932번째로 뛰어난 점수를 기록했습니다."

### Example

리더보드를 나타내는 데 정렬 집합을 사용하는 두 가지 방법이 있다. 레이서의 새로운 점수를 알고 있다면 `ZADD` 명령어를 사용하여 직접 업데이트할 수 있다. 또는 기존 점수에 점수를 추가하려면 `ZINCRBY` 명령어를 사용할 수 있다.

```bash
> ZADD racer_scores 100 "Wood" # "Wood" 레이서의 점수를 100으로 설정하여 racer_scores 정렬 집합에 추가 (1개 추가 성공)
(integer) 1
> ZADD racer_scores 100 "Henshaw" # "Henshaw" 레이서의 점수를 100으로 설정하여 racer_scores 정렬 집합에 추가 (1개 추가 성공)
(integer) 1
> ZADD racer_scores 150 "Henshaw" # "Henshaw" 레이서의 점수를 150으로 설정하여 racer_scores 정렬 집합에 추가 (이미 존재하여 추가되지 않고 점수만 업데이트)
(integer) 0
> ZINCRBY racer_scores 50 "Wood" # "Wood" 레이서의 점수를 50만큼 증가 (100+50=150)
"150"
> ZINCRBY racer_scores 50 "Henshaw" # "Henshaw" 레이서의 점수를 50만큼 증가 (150+50=200)
"200"
```

---

## Performance

대부분의 정렬 집합 연산은 `O(log(n))`이다. 여기서 n은 member의 개수이다.

수만 개 이상의 반환 값을 가진 `ZRANGE` 명령을 실행할 때는 조심해야 한다. 이 경우 시간 복잡도는 `O(log(n) + m)`이며, 여기서 m은 반환된 결과의 수이다.

---

## Sorted Set 사용 사례

그래서 Sorted Set은 언제 사용할까?

1. 해시 컬렉션의 가장 많은/적은 속성을 나열하고 싶은 경우
	- 도서 구매 애플리케이션
		- 각각의 책 정보를 개별 해시로 저장한 후,
		- 해당 책 정보들을 리스트로 조회, 리뷰/평점/구매 많은 순 등 특정 순서로 정렬하고 싶을 때 정렬 집합이 유용하다.
		- 가장 많은 리뷰순/가장 높은 평점순/가장 많은 구매순 => 각 기준에 대해 별도의 정렬 집합을 구성할 수 있다.
		- `books:reviews` -> book id를 member로, 해당 책의 리뷰 수를 score로 저장
		- 아주 간단한 ZRANGE 연산을 통해 가장 많은 리뷰가 있는 책을 조회할 수 있다. (`ZRANGE books:reviews 0 1 REV`)
		- 위에서 얻은 book id를 이용하여 `books:5` key로 해시 조회
2. 레코드간 관계를 생성하고, 여러 기준에 따라 관계를 정렬하고 싶은 경우
	- 공동 책 쓰기 애플리케이션
		- 저자, 책 각각의 해시를 저장한다.
		- 여기서 Relation 관련 정보를 추가하려면 Sorted Set을 사용하자.
		- `authors:books:4` -> member는 book id, score는 판매 부수
		- author 4가 쓴 책 중 가장 인기있는 책이 무엇일까? 이 경우 `authors:books:4` 정렬 집합의 가장 score가 높은 member의 book을 조회하면 된다.

동일한 값(예: 아이템의 조회수)을 개별 해시(예: 아이템 해시)에도 저장하고, 별도의 정렬 집합(예: 아이템의 조회수 정렬 집합)에도 복제해서 저장하는 구조이다. 이렇게 하면 낭비처럼 보일 수 있다. 하지만 괜찮다. 일반적으로 Redis를 사용하는 것은 조회를 최적화하기 위해서이기 때문이다. 어떤 아이템이 가장 높은 조회수를 가지고 있는지 아주 쉽게 찾을 수 있다. 

---

## 다이어그램 작성하기

이미 작성된 코드에 무언가를 추가하는 것은 혼란스러울 수 있다. 따라서 다이어그램을 작성하는 것이 좋다.

아래와 같은 로직을 다이어그램으로 그려놓고 관리하자.

- item 생성시
	1. item hash의 `views` 속성 0으로 초기화
	2. item id를 sorted set에 score 0으로 저장

- item 조회시
	1. item hash `views` 속성 1 증가
	2. sorted set의 item의 score 1 증가

---

## 결론

Redis 정렬 집합(Sorted Set) 자료구조는 유니크한 멤버들을 가지며, 각 멤버는 점수와 함께 저장되어 빠른 정렬 및 탐색을 제공한다. 멤버의 추가, 삭제, 점수 업데이트가 `O(log(N))` 시간 복잡도를 가지므로 대규모 데이터에서 효율적이다. 중복된 데이터를 자동으로 제거하여 메모리를 효율적으로 사용한다. 스코어를 기반으로 데이터를 정렬하고, 다양한 방식으로 정렬 순서를 조정할 수 있다.
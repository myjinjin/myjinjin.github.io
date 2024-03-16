---
layout: post
title: "[Redis] HyperLogLog 자료구조"
date: 2024-03-16 15:00:00
categories: Redis
---

---

Redis HyperLogLog는 매우 큰 데이터 집합의 중복 제거된 원소 개수를 추정하는 확률적 데이터 구조이다.

---

## HyperLogLog 특징

1. 매우 적은 메모리 (약 1.2KB)만 사용하여 수백만 개 이상의 고유한 원소를 추정할 수 있다.
2. 1% 이하의 오차율로 고유한 원소 개수를 매우 높은 정확도로 추정한다.
3. 데이터 추가 및 추정 작업을 매우 빠르게 수행한다.

---

## HyperLogLog Use Cases

- 웹사이트 방문자 분석
	- 방문자 수 추정
		- 하루, 주, 월별, 페이지별 유니크 방문자 수 추정
	- 사용자 행동 분석
		- 유니크 페이지뷰 수 추정
		- 유니크 클릭 수 추정
- 실시간 분석
	- 데이터 스트림에서 유니크 이벤트 개수 추정
		- 로그 분석: 유니크 에러 메시지 개수 추정
		- 클릭 스트림 분석: 유니크 클릭 개수 추정
		- 네트워크 트래픽 분석: 유니크 IP 주소 개수 추정
	- 실시간 대시보드
		- 실시간 방문자 수 추정
		- 실시간 트윗 수 추정
		- 실시간 주문 수 추정
- 소셜 미디어 분석
	- 특정 해시태그 사용 횟수 추정
	- 특정 사용자의 팔로워 수 추정
	- 특정 게시물의 좋아요 수 추정
- 게임 분석
	- 유니크 플레이어 수 추정
	- 유니크 게임 아이템 개수 추정
	- 게임 내 특정 이벤트 발생 횟수 추정

---

## HyperLogLog 사용시 고려 사항

- HyperLogLog는 확률적 알고리즘이므로 정확도가 100% 보장되지 않는다.
- HyperLogLog는 데이터 추가 후에는 수정할 수 없다.
- HyperLogLog는 메모리 효율적이지만, 정확도를 높이려면 더 많은 메모리를 사용해야 한다.

---

## HyperLogLog를 사용하는 실제 애플리케이션 예시

- Twitter: 트윗 수 추정
- Google Analytics: 웹사이트 방문 수 추정

---

## HyperLogLog 추정 알고리즘 원리

### 핵심 개념

- 비트 배열: 고정 길이의 비트 배열 (1.2KB 기본값)
- 해시 함수: 데이터를 비트 배열 인덱스로 매핑
- 레지스터: 최대 16개의 레지스터 (16비트)

### 추정 과정

- 데이터가 추가될 때 해시 함수를 사용하여 비트 배열 인덱스 계산
- 해당 인덱스의 비트 값을 1로 설정
- 레지스터
	- 각 레지스터는 비트 배열의 특정 부분 담당
	- 레지스터 값은 해당 부분에서 마지막으로 1로 설정된 비트의 위치 저장
- 유니크 원소 개수 추정
	- 레지스터 값과 비트 배열 길이 사용
	- Lin-Benczur 공식 사용하여 추정

### 간략 설명

- 비트 배열에서 1로 설정된 비트 개수가 많을수록 유니크 원소 개수가 많다고 추정
- 레지스터는 1로 설정된 비트의 위치 정보 저장하여 추정 정확도 향상
- Lin-Benczur 공식은 비트 배열의 특성을 고려하여 보다 정확한 추정

---

## HyperLogLog 명령어

### PFADD

- HyperLogLog에 데이터 추가

```bash
PFADD key element1 element2 ...
```

### PFCOUNT

- HyperLogLog의 유니크 원소 개수 추정

```bash
PFCOUNT key
```

### PFMERGE

- 여러 HyperLogLog를 합산

```bash
PFMERGE destkey sourcekey1 sourcekey2 ...
```

---

## Example

```bash
# 자전거 종류별 유니크 이름 추정
# 'bikes' HyperLogLog에 자전거 이름 추가
> PFADD bikes Hyperion Deimos Phoebe Quaoar
(integer) 1
# 'bikes' HyperLogLog에 있는 유니크 자전거 이름 개수 추정
> PFCOUNT bikes
(integer) 4
# 'commuter_bikes' HyperLogLog에 통근 자전거 이름 추가
> PFADD commuter_bikes Salacia Mimas Quaoar
(integer) 1
# 'bikes'와 'commuter_bikes' HyperLogLog를 merge하여 'all_bikes' HyperLogLog 생성
> PFMERGE all_bikes bikes commuter_bikes
OK
# 'all_bikes' HyperLogLog에 있는 유니크 자전거 이름 개수 추정
> PFCOUNT all_bikes
(integer) 6
```

---

## Performance

- PFADD, PFCOUNT
	- HyperLogLog에 데이터 추가(PFADD)와 추정된 유니크 개수 확인(PFCOUNT)은 모두 상수 시간과 상수 공간에서 수행된다. 즉, 데이터 크기에 상관없이 항상 빠르게 동작하며, 일정량의 메모리만 사용한다.
- PFMERGE
	- 여러 HyperLogLog를 합병하는 작업(PFMERGE)은 **O(n)**의 시간 복잡도를 가진다. 여기서 n은 합칠 HyperLogLog 개수를 나타낸다. 즉, 합칠 대상의 개수가 많아질수록 합병 시간이 증가하지만, 여전히 선형적인 속도를 유지한다.

---

## Limits

HyperLogLog 알고리즘은 최대 18,446,744,073,709,551,616 (2의 64제곱)개의 유니크한 원소를 가진 집합의 카디널리티를 추정할 수 있다. 즉, HyperLogLog는 매우 큰 집합의 유니크한 원소 개수를 대략적으로 추정할 수 있다는 의미이다. 이는 특히 메모리 사용량이 적으면서도 큰 데이터 집합을 다루어야 하는 경우에 유용하다.

물론 HyperLogLog는 완벽하게 정확한 값을 제공하지 않고 추정치를 반환하지만 일반적으로 오차 한계가 작기 때문에 대부분의 상황에 적합하다.

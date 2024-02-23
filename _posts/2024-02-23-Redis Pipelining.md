---
layout: post
title: "[Redis] Redis 파이프라이닝(Pipelining)"
date: 2024-02-22 15:00:00
categories: Redis
---

---

## Redis 파이프라이닝이란

- Redis 파이프라이닝은 Redis 명령을 <span class="important">
일괄 처리</span>하여 왕복 시간을 최적화하는 방법이다.
- 개별 명령에 대한 응답을 기다리지 않고 여러 명령을 한 번에 실행하여 <span class="important">성능을 개선</span>하는 기술이다.

---

## 요청/응답 프로토콜과 왕복 시간(RTT)

Redis는 클라이언트-서버 모델과 요청/응답 프로토콜을 사용하는 TCP 서버이다. 일반적으로 요청은 다음 단계로 이루어진다.

1. 클라이언트는 서버에 쿼리를 보내고, 보통 블로킹 방식으로 소켓에서 서버 응답을 읽는다.
2. 서버는 명령을 처리하고 응답을 클라이언트로 다시 보낸다.

예를 들어 네 개의 명령 시퀀스는 다음과 같다.
```bash
Client: INCR X
Server: 1
Client: INCR X
Server: 2
Client: INCR X
Server: 3
Client: INCR X
Server: 4
```
클라이언트와 서버는 네트워크 링크를 통해 연결된다. 링크는 매우 빠르거나(루프백 인터페이스) 매우 느릴(두 호스트 사이에 많은 홉이 있는 인터넷을 통해 설정된 연결) 수 있다. 

네트워크 지연 시간에 상관없이 패킷이 클라이언트에서 서버로 이동하고 서버에서 클라이언트로 다시 돌아와 응답을 전달하는 데는 시간이 걸린다. 이 시간을 <span class="important">왕복 시간</span>(RTT, Round Trip Time)이라고 한다.

클라이언트가 연속적으로 많은 요청을 수행해야 할 때, 예를 들어 한 리스트에 많은 항목을 추가하거나 많은 키로 데이터베이스를 채우는 경우, RTT가 성능에 어떻게 영향을 미칠 수 있는지 쉽게 이해할 수 있다. 예를 들어, RTT 시간이 250밀리초인 경우(인터넷을 통한 매우 느린 링크의 경우), 서버가 초당 100,000개의 요청을 처리할 수 있다고 하더라도, 최대 4개의 요청만 처리할 수 있다.

사용되는 인터페이스가 루프백 인터페이스인 경우, RTT는 일반적으로 밀리초 미만으로 훨씬 짧지만, 연속으로 많은 쓰기 작업을 해야하는 경우에는 이마저도 시간이 많이 늘어난다.

다행히 이 사용 사례를 개선할 수 있는 방법이 있다.

---

## Redis 파이프라이닝(Pipelining)

요청/응답 서버는 이전 응답을 클라이언트가 아직 읽지 않았더라도 새로운 요청을 처리할 수 있는 방식으로 구현될 수 있다. 이렇게 하면 클라이언트가 이전 응답을 기다리지 않고도 서버에 여러을 명령 전송하고 한 번에 응답을 읽을 수 있다.

이를 파이프라이닝이라고 한다. 이는 수십 년 동안 널리 사용되고 있는 기술이다. 예를 들어, 많은 POP3 프로토콜 구현에서는 이미 파이프라이닝 기능을 지원하고 있으며, 이를 통해 서버로부터 새 이메일을 다운로드하는 과정이 현저하게 가속화되었다.

Redis는 초기 버전부터 파이프라이닝을 지원해왔으므로, 실행 중인 버전에 관계없이 Redis와 함께 파이프라이닝을 사용할 수 있다. 다음은 netcat 유틸리티를 사용한 예시이다.

```bash
$ (printf "PING\r\nPING\r\nPING\r\n"; sleep 1) | nc localhost 6379
+PONG
+PONG
+PONG
```

이번에는 각 호출마다 RTT 비용을 지불하는 것이 아니라 세 개의 명령에 대해 한 번만 RTT 비용을 지불한다.

명확하게 설명하자면, 파이프라이닝을 사용하면 첫 번째 예제의 작업 순서는 다음과 같다.

```bash
Client: INCR X
Client: INCR X
Client: INCR X
Client: INCR X
Server: 1
Server: 2
Server: 3
Server: 4
```

> 참고💡
>
>클라이언트가 파이프라이닝을 사용하여 명령을 전송하는 동안, 서버는 메모리를 사용하여 응답을 강제로 대기열에 넣어야 한다. 따라서 파이프라이닝을 사용하여 많은 명령을 보내야 하는 경우에는 각각 적절한 수(예: 10,000개의 명령)를 포함하는 일괄 처리를 보내고, 응답을 읽은 다음 다시 10,000개의 명령을 보내는 식으로 보내는 것이 좋다. 속도는 거의 동일하지만 추가로 사용되는 메모리는 이 10,000개의 명령에 대한 응답을 대기열에 대기시키는 데 필요한 양만큼만 늘어난다.

---

## RTT만이 문제가 아니다!

파이프라이닝은 왕복 시간과 관련된 지연 비용을 줄이는 방법일 뿐만 아니라, 실제로 주어진 Redis <span class="important">서버에서 초당 수행할 수 있는 작업 수를 크게 향상</span>시킨다. 파이프라이닝을 사용하지 않을 때, 각각의 명령을 처리하는 데는 데이터 구조에 접근하고 응답을 생성하는 측면에서는 비교적 저렴하지만, 소켓 I/O 측면에서는 매우 비용이 많이 들게 된다. 이 작업에는 `read()`와 `write()` 시스템 호출이 포함되어 있으며, 이는 유저 영역과 커널 영역 간의 전환을 의미한다. 이 과정에서 발생하는 컨텍스트 스위치는 속도를 현저하게 저하시킨다.

파이프라이닝을 사용하면 일반적으로 여러 명령이 하나의 `read()` 시스템 호출로 한꺼번에 읽히고, 여러 응답이 하나의 `write()` 호출로 전송된다. 그 결과, 초기에는 파이프라인이 길어질수록 초당 총 쿼리 수가 거의 선형적으로 증가하고, 결국 아래 그림과 같이 파이프라이닝을 사용하지 않았을 때보다 10배 가량 높은 성능을 얻을 수 있다. 

![redis06-pipeline_iops](/assets/images/redis06-pipeline_iops.png)

---

## Code Example in Go

파이프라이닝을 지원하는 Redis Go 클라이언트를 사용하여 파이프라이닝으로 인한 속도 향상을 테스트해보았다.

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func bench(descr string, fn func()) {
	start := time.Now()
	fn()
	fmt.Printf("%s %v seconds\n", descr, time.Since(start))
}

func withoutPipelining() {
	rdb := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})
	for i := 0; i < 10000; i++ {
		rdb.Ping(ctx)
	}
}

func withPipelining() {
	rdb := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})
	pipe := rdb.Pipeline()
	for i := 0; i < 10000; i++ {
		pipe.Ping(ctx)
	}
	_, err := pipe.Exec(ctx)
	if err != nil {
		panic(err)
	}
}

func main() {
	bench("without pipelining", withoutPipelining)
	bench("with pipelining", withPipelining)
}
```

실행 결과는 다음과 같다.

```bash
 go run main.go
without pipelining 668.024715ms seconds
with pipelining 9.112195ms seconds
```

파이프라이닝을 사용하여 전송 속도를 향상시켰다!

---

## 결론

Redis 파이프라이닝(Pipelining)은 클라이언트가 모든 명령을 한 번에 보내고, 그 후에 일괄적으로 모든 응답을 받기 때문에 네트워크 오버헤드를 줄이고 처리 속도를 향상시킬 수 있다. 이를 통해 대규모 데이터 처리 및 응용 프로그램 성능을 향상시킬 수 있다. 그러나 응답 순서가 보장되지 않고, 서버의 메모리 사용량이 늘어날 수 있으므로 주의가 필요하다.

---

- Reference: https://redis.io/docs/manual/pipelining/
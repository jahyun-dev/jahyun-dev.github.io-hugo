Kafka Producer 설정 시 고려사항

### ACKS

얼마나 많은 파티션으로부터 성공 응답을 받아야 메시지 전송 성공으로 처리할 것인가? 메시지 유실 가능성에 큰 영향을 미친다.

- 0 : 기다리지 않고 바로 성공 처리
- 1 : 리더로부터 성공 받으면 바로 성공처리
- all : 모든 복제본으로부터 성공 메시지를 받아야 함

### BUFFE.MEMORY

메시지 버퍼 메모리 설정. 

### RETRIES

얼마나 많이 retry 할까? 기본적으로 프로듀서는 기다린다 리트라이 사이 100ms를. retry.backoff.ms를 사용해 조절할 수 있다. 


### Ttimeout.ms, request.timeout.ms, metadata.fetch.timeout.ms



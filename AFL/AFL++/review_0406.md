# AFL++

## 1. cull_queue()
edge(coverage slot)를 기준으로 seed를 평가하고, 전체 coverage를 대표하는 최소한의 seed 집합만 favored로 선택

```c
void cull_queue(afl_state_t *afl) {

  if (likely(!afl->score_changed || afl->non_instrumented_mode)) { return; }
  // score_changed == 0 : 점수 변화 없음
  // non_instrumented_mode : 커버리지 추적 안되는 모드

  u32 len = (afl->fsrv.map_size >> 3);  // 커버리지 비트맵 크기 (8배 해서 바이트 단위로 변환) 
  u32 i;
  u8 *temp_v = afl->map_tmp_buf;  // 아직 커버되지 않은 영역을 추적하는 임시 비트맵 

  afl->score_changed = 0;

  memset(temp_v, 255, len);  // 모든 비트 1로 초기화 (아직 아무 커버리지도 선택되지 않음) 

  afl->queued_favored = 0;
  afl->pending_favored = 0;

  for (i = 0; i < afl->queued_items; i++) {  // favored 초기화 (아직 favored 시드 없음) 
    afl->queue_buf[i]->favored = 0;
  }

  /* Let's see if anything in the bitmap isn't captured in temp_v.
     If yes, and if it has a afl->top_rated[] contender, let's use it. */

  afl->smallest_favored = -1;

  for (i = 0; i < afl->fsrv.map_size; ++i) {  // 커버리지 비트맵의 각 비트(엣지) 순회 
    if (afl->top_rated[i] && (temp_v[i >> 3] & (1 << (i & 7))) &&  // afl->top_rated[i] : 이 커버리지 비트를 가장 잘 만든 시드 존재
        afl->top_rated[i]->trace_mini) {  // temp_v[...] & ... : 아직 이 커버리지 비트는 선택되지 않음 / trace_mini : 이 시드의 커버리지 정보 있음 

      u32 j = len;

      /* Remove all bits belonging to the current entry from temp_v. */
      while (j--) { // 커버리지 제거 : 이 시드가 커버하는 영역을 temp_v에서 제거 (중복 커버리지 제거) 
        if (afl->top_rated[i]->trace_mini[j]) {
          temp_v[j] &= ~afl->top_rated[i]->trace_mini[j];
        }
      }

      if (!afl->top_rated[i]->favored && !afl->top_rated[i]->disabled) {  // 아직 선택 안 됐고 비활성도 아니면 favored로 채택 
        afl->top_rated[i]->favored = 1;
        ++afl->queued_favored;

        if (!afl->top_rated[i]->was_fuzzed) {  // 아직 퍼징된 적 없는 시드 
          ++afl->pending_favored;  // 퍼징 안 한 favored seed 개수 추가 (pending = favored한데 아직 퍼징 안 한 시드 수) 
          if (unlikely(afl->smallest_favored < 0 || 
                       afl->smallest_favored > (s64)afl->top_rated[i]->id)) {
            afl->smallest_favored = (s64)afl->top_rated[i]->id;  
          }
        }
      }
    }
  }

  for (i = 0; i < afl->queued_items; i++) {   
    if (likely(!afl->queue_buf[i]->disabled)) {  
      mark_as_redundant(afl, afl->queue_buf[i], !afl->queue_buf[i]->favored);  // favored == 0 이면 redundant 처리
    }
  }
  afl->reinit_table = 1;  // 내부 테이블 다시 초기화 필요
}
```
- `afl->top_rated[i]` : coverage 비트맵의 i번째 위치를 가장 잘 대표한다고 현재 판단된 seed
- smallest_favored : 아직 fuzz되지 않은 favored seed 들 중에 가장 작은 id를 저장
  - 매번 전체 queue 다 찾으면 느리니까 최소 id 하나 기억해두고 거기부터 탐색 시작
- 전체 흐름
  - (1) 실행 -> coverage 수집
    - 시드 하나 실행 -> 실행 중 edge 발생 -> coverage bitmap에 기록됨
  - (2) top_rated 생성
    - 각 커버리지 위치 i마다 이 edge를 가장 잘 대표하는 시드 하나만 선택 
    - ex. edge 10 : 시드 A (빠름), 시드 B (느림) --> `top_rated[10] = A`
  - (3) cull_queue : edge(coverage slot)를 기준으로 seed를 평가하고, 전체 coverage를 대표하는 최소한의 seed 집합만 favored로 선택
    - 1. 초기화 : temp_v를 모두 1(미탐색)으로 세팅
      2. 커버리지 맵을 처음부터 끝까지 순회
         - temp_v에서 아직 1인 엣지를 발견하면, 해당 엣지의 top_rated 시드를 favored로 설정
         - **중복 엣지 제거** : 방금 뽑힌 시드가 커버하는 모든 엣지를 `temp_v`에서 한 번에 `0`으로 지움 --> **greedy Algorithm** : 최소한의 seed로 전체 coverage 대
      3. 갱신
         - favored 시드 중 아직 퍼징되지 않은 시드 개수 카운트
         - 그 중 가장 앞번호 시드를 다음 퍼징 1순위로 기록
      4. redundant 처리
         - 큐를 다시 훑어 끝까지 favored = 0인 시드를 redundant로 처리
         - 명단이 바뀌었으므로 reunut_table = 1로 세팅하여 alias table 재계산 지

## 2. perf_score
- `perf_score` : 시드 구조체의 멤버 변수로 저장 
  - perf_score 형성 자체는 비선형적
  - 지수, 로그 등의 계산이 들어감
- `perf_score`를 바탕으로 `alias table`에서 해당 시드가 선택될 확률은 선형적임
  ```
  ``` 
  - ex. 시드 A의 perf_score : 200 / 시드 B의 perf_score = 100 --> alias probablity에서 시드 A가 선택될 확률은 시드 B의 2배
  




## 3. trim_case_custom()
- `trim_case_custom()`의 defalut : 기존 AFL 함수의 `trim_case()`와 동일함
- 근데 가장 큰 차이점 : trim을 거친 결과물이 원본보다 크기가 커져도 허용함 (AFL은 파일 크기 줄이는 것이 목표)
  - 허용 이유 : Grammer 기반 퍼징 등에서 논리적 구조를 단순화하는 과정에서 바이트 크기가 오히려 늘어날 수 있는 케이스를 고려하여 이를 허용하도록 설계되어 있기 때문
  - 커버리지가 원본과 일치해야 한다는 전제는 동일하게 유지 


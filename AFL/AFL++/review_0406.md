# AFL++

## 1. cull_queue()
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
      while (j--) {
        if (afl->top_rated[i]->trace_mini[j]) {
          temp_v[j] &= ~afl->top_rated[i]->trace_mini[j];
        }
      }

      if (!afl->top_rated[i]->favored && !afl->top_rated[i]->disabled) {
        afl->top_rated[i]->favored = 1;
        ++afl->queued_favored;

        if (!afl->top_rated[i]->was_fuzzed) {
          ++afl->pending_favored;
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
      mark_as_redundant(afl, afl->queue_buf[i], !afl->queue_buf[i]->favored);
    }
  }
  afl->reinit_table = 1;
}
```


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


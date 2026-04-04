# AFL++

## AFL++ 코드 분석 
1. `src/afl-fuzz.c`   
- 역할 : 퍼저 전체 시작점 (main 함수 존재)
- 기능
  - 퍼징 환경 설정 및 초기화 (`afl_state_init()`, `afl_fsrv_init()` 등의 초기화 함수 존재)
  - 옵션 / 인자 처리 
  - 전체 퍼징 실행 흐름 조율 
- 특징
  - 핵심 퍼징 세부 로직은 다른 파일로 분리되어 있음
  - 이 파일은 전체 제어 중심에 가깝고, 세부 mutation, queue, trim 구현의 중심 파일은 아님  

2. `src/afl-fuzz-one.c`
- 역할 : 한 개의 시드에 대해 실제 퍼징 수행
- **Calibration** 단계 존재
  - queue_cur의 cal_failed 상태를 보고 calibrate_case() 재호출
  - AFL의 calibrate 단계가 AFL++에도 유지됨

- **trimming** 단계 존재 
  - `trim_case()` 호출 확인   
    `u8 res = trim_case(afl, afl->queue_cur, in_buf);`
  - 조건
    ```
    if (unlikely(!afl->non_instrumented_mode &&
             !afl->queue_cur->trim_done &&
             !afl->disable_trim)) {
    ```
    - 무조건 trim 하는 게 아니라, 조건을 만족할 때만 trim 함
    - non-instrumented mode가 아니고, 아직 trim 안 했고, trim 비활성화 옵션도 아닐 때
    - AFL의 trim 단계가 유지되되 조건 제어가 더 명시적임

- **Performance score** 단계 존재
  - `calculate_score()` 호출 확인   
    `calculate_score(afl, afl->queue_cur);`
  - old_seed_selection 여부에 따라 perf_score 처리 방식이 다름
    ```
    if (likely(!afl->old_seed_selection))
      orig_perf = perf_score = afl->queue_cur->perf_score; // 이미 계산되어 저장된 perf_score 사용 
    else
      afl->queue_cur->perf_score = orig_perf = perf_score = calculate_score(afl, afl->queue_cur);
    ```

- **deterministic** 단계 존재
  - deterministic mutation 단계의 비용이 크기 때문에, 모든 시드에 대해 이를 수행하지 않음
  - skip 하는 경우
    - skip_deterministic 플래그가 켜진 경우
    - 현재 시드가 이미 deterministic testing을 마친 경우(passed_det)
    - quick effective bytes 정보상 deterministic mutation의 효율이 낮다고 판단되는 경우
    - 현재 시드의 perf_score가 낮아 투자 가치가 작다고 판단되는 경우
  -->  deterministic stage를 무조건 수행하는 대신, 유망한 시드에만 자원을 집중하도록 최적화함 (AFL에 비해 스킵 조건이 조금 더 많음) 
  
- **havoc stage** 존재
  - 다양한 mutation operator를 무작위로 적용하며, 반복 횟수(stage_max)는 현재 시드의 perf_score에 따라 조정됨
  - 따라서 더 유망한 시드는 더 많은 mutation 기회를 받음 

- **splicing stage** 존재
  - 역할 : 새 path 발견이 잘 안 될 때 사용하는 fallback 전략
  - 조건
    ```
    if (afl->use_splicing && splice_cycle++ < SPLICE_CYCLES &&
    afl->ready_for_splicing_count > 1 && afl->queue_cur->len >= 4)
    ```
    - `afl->use_splicing` : splicing 기능이 켜져 있어야 함
    - `splice_cycle++ < SPLICE_CYCLES` : splice를 너무 무한정 반복하지 않음
    - `afl->ready_for_splicing_count > 1` : 섞을 만한 다른 시드가 최소 1개는 있어야 함
    - `afl->queue_cur->len >= 4` : 현재 입력이 너무 짧으면 splice 의미가 없음  
  --> 제한된 조건에서만 실행됨
  - 다른 queue entry 하나 고르기
    ```
    do {
      tid = rand_below(afl, afl->queued_items);
    } while (
      unlikely(tid == afl->current_entry || afl->queue_buf[tid]->len < 4));
    ```
    - queue 안에서 랜덤하게 다른 시드 하나 뽑음
  - 두 입력이 어디서 다른지 찾음
    ```
    locate_diffs(in_buf, new_buf, MIN(len, (s64)target->len), &f_diff, &l_diff);
    ```
    - 현재 입력과 선택된 다른 입력을 비교해서 처음으로 다른 위치, 마지막으로 다른위치를 찾음 (두 입력이 실제로 다른 구간의 범위를 찾음)
  - 차이가 너무 작으면 다시 시도
    ```
    if (f_diff < 0 || l_diff < 2 || f_diff == l_diff) { goto retry_splicing; }
    ```
  - 자를 위치 결정
    ```
    split_at = f_diff + rand_below(afl, l_diff - f_diff);
    ```
    - 처음 다른 바이트와 마지막 다른 바이트 사이 어딘가를 랜덤하게 골라서 거기를 splice 경계로 삼음
  - 실제로 두 입력을 섞음
    ```
    memcpy(afl->in_scratch_buf, in_buf, split_at);
    memcpy(afl->in_scratch_buf + split_at, new_buf + split_at, len - split_at);
    ```
  - 새 버퍼를 현재 입력으로 교체
    ```
    in_buf = afl->in_scratch_buf;
    ...
    memcpy(out_buf, in_buf, len);
    ```
    - 방금 splice해서 만든 입력을 이제 다음 mutation의 기준 입력으로 사용함
  - 어디로 감?
    ```
    goto custom_mutator_stage;
    ```
    - splice가 끝났다고 바로 실행하는 게 아니라, 이 결과물을 다음 mutation 단계로 넘김

  --> splice → custom mutator stage → (이후) havoc 계열 변형 → 실행



### trim_case() 함수 구현 차이
1. trim_case()의 목적
- 입력을 최소화하는 함수  
- 아이디어 : 입력의 일부를 삭제했을 때 프로그램 실행 경로(trace)가 변하지 않으면, 해당 부분은 불필요하다고 판단하고 제거함  

2. AFL trim() 동작 구조
- (1) 입력 길이 확인 (`q->len < 5` → 종료)  
- (2) 삭제할 chunk 크기 결정 (`remove_len`)
- (3) 반복
  - 입력 일부 삭제
  - `run_target()` 실행
  - 기존 실행과 동일하면 -> 삭제 유지
- (4) chunk 크기를 점점 줄여가며 반복
```
write_with_gap(in_buf, q->len, remove_pos, trim_avail);

fault = run_target(argv, exec_tmout);

cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST);

if (cksum == q->exec_cksum) {
    // 삭제 유지
    q->len -= trim_avail;
    memmove(...);
}
```

3. AFL++ trim_case() 동작 구조
- (1) Custom Mutator 기반 trimming 지원
  ```
  if (afl->custom_mutators_count) {
    trim_case_custom(afl, q, in_buf);
  }
  ```
  - 사용자 정의 mutator가 trimming 수행 가능  
  - AFL은 고정된 trimming 방식  
  - AFL++는 플러그인 기반 확장 가능
  - AFL : 고정 알고리즘 / AFL++ : 사용자 정의 가능   
  - Why? --> 특정 포맷(PDF, PNG)은 구조-aware trimming이 더 효과적

- (2) testcase 재동기화
  ```
  if (orig_len != q->len) {
    queue_testcase_retake(afl, q, orig_len);
  }
  ```
  - trimming 이후 testcase 길이, 내용 변경됨  
  - queue 내부 상태와 불일치 발생 가능 -> 이를 다시 맞춤
  - AFL : 명시적인 재동기화 없음 (구조 자체가 단순해서 괜찮음) / AFL++ : trimming 후 queue 재정리 
  - Why? --> queue 기반 seed 관리가 더 복잡하기 때문에 consistency 유지가 필요

- (3) 상태 관리 구조 변화
  - AFL : `trim_case(argv, q, buf)` --> 전역 변수 중심
  - AFL++ : `trim_case(afl_state_t *afl, q, buf)` --> afl_state_t 구조체 기반

- (4) trim 수행 조건 (호출 조건 차이)
  AFL++ :
  ```
  if (!afl->non_instrumented_mode &&
    !q->trim_done &&
    !afl->disable_trim) {
    trim_case(...);
  }
  ```
  - trim을 수행할지 명시적으로 제어
  - 상황에 따라 trimming 비활성화 가능
  - AFL은 단순 조건 / AFL++는 다양한 실행 조건 존재



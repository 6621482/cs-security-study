# ROP

## ROP
- **ROP** : 리턴 가젯을 이용하여 복잡한 실행 흐름을 구현하는 기법
  - 상황에 맞춰 return to library, return to dl-resolve, GOT overwrite 등의 페이로드를 구성함
- ROP chain : ret 단위로 여러 코드가 연쇄적으로 실행되는 것
  
### exploit 흐름 정리 
1. 카나리 우회
2. system 함수 주소 계산
   - GOT에 등록된 함수를 이용해서 libc.so.6이 매핑된 영역의 주소를 구할 수 있음
   - 오프셋 구하는 방법
     `read -s <파일명> | grep " <함수명>@"`
3. "/bin/sh"
   - 다른 파일(libc.so.6)에 포함된 문자열 사용
     - libc 영역의 임의 주소를 구하고, 그 주소로부터 오프셋을 더하거나 빼서 계산함
   - 임의 버퍼에 직접 주임하여 참조
4. GOT Overwrite

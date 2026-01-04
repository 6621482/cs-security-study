# CS Basics
Computer science fundamentals for DreamHack Beginners.

## Data representation

### Bit & Byte
- **Bit** : 0 또는 1의 값을 가지는 데이터의 최소 단위
- **Byte** : 8개의 비트로 구성된 더 큰 단위, 메모리에 저장되는 최소단위

### MSB & LSB
- **MSB**(Most Significant Bit)
  - 가장 왼쪽에 있는 비트 -> 숫자 크기에 가장 큰 영향
  - 부호가 있는 데이터의 경우 : MSB = 부호 비트
    - 0 : 양수 / 1 : 음수
- **LSB**(Least Significant Bit)
  - 가장 오른쪽에 있는 비트

### Byte Ordering
- **Big Endian**
  - 가장 상위 바이트가 낮은 메모리 주소에 저장됨
  - 사용 예 : 네트워크 상에서 데이터를 전송할 때, SPARC CPU
- **Little Endian**
  - 가장 하위 바이트가 낮은 메모리 주소에 저장
  - 사용 예 : Intel의 x86, x86-64 CPU
    
- **Example** : 0x01234567
  - 빅엔디안 : 0x01 | 0x23 | 0x45 | 0x67
  - 리틀엔디안 : 0x67 | 0x45 | 0x23 | 0x01

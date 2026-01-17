---
title: "[블록 암호화] KW 알고리즘"
excerpt: "Key Wrapping Algorithm"
date: "2026-01-17"
categories: [crypto]
---

`KW`는 일반 데이터를 암호화하는 알고리즘과는 다르게  
키를 안전하게 보호하기 위한 전용 알고리즘입니다.

주로 데이터를 암호화하기 위한 키를 KEK(Key Encryption Key)로 보호합니다.

- `RFC-3394`에 KW 알고리즘에 대해 서술되고, NIST의 `SP 800-38F`를 활용하여 테스트할 수 있습니다.

# 1. KW 알고리즘이란?

| 특징          | 설명                               |
| :------------ | :--------------------------------- |
| 암호 알고리즘 | AES                                |
| 지원 키 길이  | 128, 192, 256 비트 (ECB와 동일)    |
| 입력 길이     | 최소 16 바이트 이상, 8 바이트 배수 |
| 출력 길이     | 입력 + 8 바이트                    |

# 2. KW 동작 원리

KW 알고리즘은 내부적으로 ECB 모드를 활용한 키 전용 암호화 알고리즘입니다.

Default IV인 `0xA6A6A6A6A6A6A6A6`을 고정으로 사용하며,  
출력은 내부 동작에 따라 8바이트가 추가적으로 붙습니다.

8바이트의 블록으로 나누어 연산하며, 총 블럭 갯수에 따라 반복 횟수가 증가합니다.

- 6 \* N (N은 블록 갯수)

<br/>

`RFC-3394`에 따르면 아래와 같이 작성되어 있습니다.

## 2-1. Key Wrap

```txt
 Inputs:      Plaintext, n 64-bit values {P1, P2, ..., Pn}, and
                Key, K (the KEK).
   Outputs:     Ciphertext, (n+1) 64-bit values {C0, C1, ..., Cn}.

   1) Initialize variables.

       Set A0 to an initial value (see 2.2.3)
       For i = 1 to n
            R[0][i] = P[i]

   2) Calculate intermediate values.

       For t = 1 to s, where s = 6n
           A[t] = MSB(64, AES(K, A[t-1] | R[t-1][1])) ^ t
           For i = 1 to n-1
               R[t][i] = R[t-1][i+1]
           R[t][n] = LSB(64, AES(K, A[t-1] | R[t-1][1]))

   3) Output results

       Set C[0] = A[t]
       For i = 1 to n
           C[i] = R[t][i]
```

## 2-2. Key Unwrap

```txt
 Inputs:  Ciphertext, (n+1) 64-bit values {C0, C1, ..., Cn}, and
           Key, K (the KEK).
   Outputs: Plaintext, n 64-bit values {P1, P2, ..., Pn}.

   1) Initialize variables.

       Set A[s] = C[0] where s = 6n
       For i = 1 to n
           R[s][i] = C[i]

   2) Compute intermediate values.

       For t = s to 1
           A[t-1] = MSB(64, AES-1(K, ((A[t] ^ t) | R[t][n]))
           R[t-1][1] = LSB(64, AES-1(K, ((A[t]^t) | R[t][n]))
           For i = 2 to n
               R[t-1][i] = R[t][i-1]

   3) Output results.

       If A[0] is an appropriate initial value (see 2.2.3),
       Then
           For i = 1 to n
               P[i] = R[0][i]
       Else
           Return an error
```

# 3. 장점과 단점

## 3-1. 장점

"키 보호 전용"으로 설계되어 아래와 같은 장점을 가질 수 있습니다.

- AIV 검증으로 무결성 내장
- 구현 단순

## 3-2. 단점

- 입력 길이 제약 (16바이트 이상의 8바이트 배수)
- 범용성 제약 (일반 데이터가 아닌 키 전용)

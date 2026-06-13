## [Assignment 02] -한국어 감성 Chatbot 구현

- Seq2Seq + Decoding Strategy 비교
- Dataset: Chatbot_data_for_Korean v1.0

---

### 0. 실험 목표
1. Greedy vs Beam Search 알고리즘 비교
2. Beam Width(3/5/7/10)의 최적점 찾기
3. 챗봇 도메인에 맞는 최적 디코딩 전략 탐색

---

### 1. 챗봇 파이프라인

| STEP | 내용 |
|------|------|
| 1 | 데이터 로드 (Q/A) |
| 2 | SentencePiece로 subword 토크나이저 학습 |
| 3 | 문장 → token ID 변환 |
| 4 | Q→encoder 입력 / A→decoder 입력&target 분리 |
| 5 | 패딩 (길이 맞춤) |
| 6 | Embedding (정수→벡터) |
| 7 | [학습] Encoder: Embedding→LSTM→(h,c) |
| 8 | [학습] Decoder: 정답 입력→LSTM→Dense |
| 9 | 모델 학습 (loss 계산, weight 업데이트) |
| 10 | [추론] Encoder: 입력→(h,c) |
| 11 | [추론] Decoder: 이전 출력+상태→다음 단어 |
| 12 | Decoding (Greedy/Beam Search) |
| 13 | 응답 생성 (BOS→EOS 반복) |
| 14 | 평가 (BLEU/속도/정성평가) |

---

### 2. 코드 개선 과정 (Greedy 기준)

| 수정 내용 | BLEU |
|-----------|------|
| 기본 베이스라인 | 0.004 |
| decode_greedy int 수정 | 0.004 |
| 하이퍼파라미터 조정 + UNK 토큰 방지 | 0.005 |
| Early Stopping (patience=15) | 0.01 |

---

### 3. 결과 비교 (Greedy vs Beam Search)

| 전략 | BLEU | 속도(ms) | 자연스러움 | 다양성 | 일관성 |
|------|------|----------|-----------|--------|--------|
| Greedy | 0.0144 | **732.3** | 1/5 | 0/5 | 1/5 |
| Beam-3 | 0.0158 | 1524.6 | 2/5 | 3/5 | 2/5 |
| Beam-5 | 0.0198 | 2472.2 | 3/5 | 2/5 | 2/5 |
| Beam-7 | **0.0363** | 17496.7 | 3/5 | 2/5 | 2/5 |
| Beam-10 | 0.0276 | 24226.9 | 3/5 | 3/5 | 2/5 |

---

### 4. 분석

💠**Greedy vs Beam**
- Beam Search가 전반적으로 BLEU 높음 (여러 후보 시퀀스를 동시 탐색)
- 단, Greedy가 속도는 압도적으로 빠름 (Beam-3: 2배, Beam-5: 3배, Beam-7: 24배, Beam-10: 33배 느림)

<br>

💠**Beam Width 최적점**
- Width 3→5→7로 갈수록 BLEU 증가, **Width=7(0.0363)에서 최고점**
- Width=10에서는 오히려 감소 → 후보가 너무 많아지면 저품질 경로까지 탐색에 포함되어 품질 저하

<br>

💠**속도 trade-off**
- Width가 커질수록 후보 수 증가 → 메모리/연산량 증가 → 속도 급격히 저하
- Width 7부터는 실용성 떨어질 정도로 느림 (17~24초)

---

### 5. 한계 및 향후 과제

<br>

> **BLEU 지표의 한계**: 단어 겹침 비율 기반이라, 의미적으로 적절한 응답도 낮은 점수를 받을 수 있음
> 
> Ex) "1지망 학교 떨어졌어.." → 정답 "위로해 드립니다" / 예측 "마음이 아프시겠어요" → 의미상 적절하나 BLEU는 거의 0

  <br>
  
- BLEU는 번역처럼 정답이 명확한 task에 더 적합
- 추후 ROUGE, **PPL**(자연스러움), **Distinct-1/2**(다양성) 등 추가 지표 평가 필요
- 전반적으로 BLEU 수치가 낮아 추가적인 코드 수정 및 학습 필요

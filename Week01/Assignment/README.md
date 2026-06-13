## [Assignment01] - NSMC LSTM 감성 분석

- Bidirectional LSTM으로 네이버 영화 리뷰 감성 분석 구현
- Test Accuracy 85% 이상 달성 목표
- Python, TensorFlow/Keras, KoNLPy(Okt), NSMC 사용



---

### 1. 데이터셋
- NSMC (training 150,000건, test 50,000건)


---

### 2. 전처리 파이프라인

| STEP | 내용 | 설명 |
|------|------|------|
| 1 | 결측치 제거 | document 컬럼 NaN/빈 문자열 제거 |
| 2 | 특수문자 제거 | 한글·공백만 유지 |
| 3 | 형태소 분석 | Okt.pos() - Noun/Verb/Adjective만 추출 |
| 4 | 길이 필터링 | 형태소 1개 이하 문장 제거, CSV로 저장 |
| 5 | Tokenizer | num_words=30000, maxlen=100 |

---

### 3. 성능 향상 전략
- 형태소 품사 필터링 (Noun/Verb/Adjective)
- 단방향 → Bidirectional LSTM
- MAX_LEN 최적화
- ReduceLROnPlateau로 학습률 스케줄링


---

### 4. 결과 비교
> 5.2는 모델 크기 증가로 overfitting이 심해져 정확도가 하락한 것으로 추정, 최종 미반영

| 실험 | 내용 | Test Acc | Test Loss |
|------|------|----------|-----------|
| Baseline | 기본 모델 | 84.05% | 0.3576 |
| 5.1 (반영) | 불용어+1글자 제거 | 84.20% | 0.3563 |
| 5.2 (미반영) | 임베딩 128→256 | 84.12% | 0.3557 |
| 5.3 (반영) | LSTM 유닛 ↑ + batch 64 | 84.39% | 0.3531 |
| 5.4 (반영) | Dropout 0.3→0.4 | **84.99%** | 0.3429 |




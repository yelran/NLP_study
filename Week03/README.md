## [Assignment 03] - Transformer Attention Head 역할 분화

- **Dataset**: opus100 Portuguese → English (학습 5,000개 / 검증 500개)
- **Model**: Transformer (d_model=96, num_layers=3, num_heads=6)
- **Framework**: TensorFlow 2.x / Google Colab GPU

---

### 실험 구성
- 실험 A: Attention Head 패턴 시각화
- 실험 B: Head 수 변화에 따른 성능 비교
- 실험 C: Head Pruning

---

### 전체 파이프라인

| STEP | 내용 |
|------|------|
| 1 | 환경 설정 (TensorFlow / tensorflow-text / nltk, GPU 확인) |
| 2 | 데이터 로드 (TED Talks pt→en, 서브워드 토크나이저) |
| 3 | 데이터 파이프라인 (Teacher Forcing, shuffle→batch→prefetch) |
| 4-5 | Positional Encoding/Embedding (sin/cos, √d_model 스케일링) |
| 7 | Attention Layers (Global/Causal Self-Attention, Cross-Attention) |
| 8 | Feed Forward Network (Dense→ReLU→Dense + Dropout + 잔차연결) |
| 9-10 | Encoder/Decoder 레이어 구성 |
| 11 | Transformer 조립 (Encoder+Decoder+Final Dense) |
| 12 | 학습 (masked_loss/accuracy, CustomSchedule, Adam) |
| 13 | Translator (Auto-regressive 생성) |
| 14-16 | 실험 B 자동화 (num_heads=1/2/6/12 비교) |
| 17 | 실험 A 시각화 (Head별 Attention heatmap) |
| 18 | 실험 C Pruning (Head 마스킹 → val_loss 변화 측정) |
| 19 | 결과 저장 (experiment_results.json) |

---

### 실험 B: Head 수 변화에 따른 성능 비교

> RAM 부족으로 세션 다운 → 샘플 수(5000→3000, 500→300), epoch(30→15) 축소 후 재실행

| Head 수 | head_dim | BLEU | 소요 시간 |
|---------|----------|------|-----------|
| 1 | 96 | **0.0218** | 874.6s |
| 2 | 48 | 0.0160 | 839.7s |
| 6 | 16 | 0.0153 | 813.9s |

**예상과 다른 결과**: Head 수가 많을수록 성능이 좋을 것으로 예상했으나, 반대로 나타남
- Head 수가 적을수록 BLEU가 높음 → 소규모 데이터셋에서는 단일 Head가 더 효과적 (head_dim이 클수록 표현력 ↑)
- Head 수가 클수록 소요 시간은 감소 → 병렬 연산 효율 때문으로 추정

---

### 실험 A: Attention Head 패턴 시각화

#### 🔹Entropy 분석 (분산↔집중 패턴)
- 전 Layer에서 Entropy 값이 2.1~2.4로 매우 유사
- → **Head 간 역할 분화가 명확하지 않음**

#### 🔹Head 간 코사인 유사도
- 모든 Head 쌍에서 유사도 0.97 이상 (최고 0.997 / 최저 0.976)
- → **6개 Head가 거의 동일한 Attention 패턴 학습** (소규모 데이터셋에서 정보 중복 학습으로 추정)

---

### 실험 C: Head Pruning

<img width="1439" height="504" alt="image" src="https://github.com/user-attachments/assets/f8094188-bed1-4de4-aa21-3769d654ba4d" />


#### Baseline
| val_loss | val_acc |
|----------|---------|
| 5.9719 | 0.1456 |

#### Head 중요도
- **Layer 0 Head 0**: val_loss를 +0.1353 상승시키는 **유일하게 중요한 Head**
- 나머지 17개 Head는 영향이 거의 없음 (±0.005 이내)

#### 누적 Pruning 결과 (주요 지점)

| 제거 Head 수 | val_loss | val_acc | baseline 대비 |
|--------------|----------|---------|----------------|
| 0개 (baseline) | 5.9719 | 0.1456 | — |
| **8개 제거 (44%)** | **5.9393** | **0.1547** | loss ↓0.55% / acc ↑6.25% |
| 17개 제거 (94%) | 5.9585 | 0.1461 | loss ↓0.22% / acc ↑0.34% |
| 18개 전부 제거 | 6.0441 | 0.1269 | loss ↑1.21% / acc ↓12.84% |

- 8개 제거 시 정확도 가장 높음 (+6.25%)
- 17개(94%) 제거해도 baseline 대비 성능 유지/향상
- → **불필요한 Head 제거 → 경량화 가능성 확인**

---

### 결론

- 소규모 데이터셋에서 Multi-Head Attention의 다수 Head는 중복 학습되는 경향
- 불필요한 Head는 제거해도 성능 유지 가능 (Head Pruning 효과적)
- 논문 「Are Sixteen Heads Really Better Than One? (NeurIPS 2019)」의 주장과 일치하는 결과
  - 대부분의 Head는 중복
  - 중요한 Head는 학습 초기에 결정됨

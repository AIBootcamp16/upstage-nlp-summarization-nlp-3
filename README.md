<!-- Header -->
<div align="center">

![header](https://capsule-render.vercel.app/api?type=rounded&height=170&text=Dialogue%20Summarization&desc=%EC%9D%BC%EC%83%81%20%EB%8C%80%ED%99%94%20%EC%9A%94%EC%95%BD%20NLP%20%EB%8C%80%ED%9A%8C%20%EC%84%A0%ED%98%95%20%EB%AA%A8%EB%8D%B8&fontSize=35&descSize=16&descAlignY=65&color=gradient&fontColor=ffffff&animation=fadeIn)

<h3>🗣️ KoBART 기반 일상 대화 요약 모델 · Dialogue Summarization Competition 🗣️</h3>

</div>

---

## 💻 프로젝트 소개

### 📌 프로젝트 개요
이 프로젝트는 **일상 대화(dialogue)를 입력으로 받아 한 문장 요약(summary)을 생성하는 NLP 모델**을 구축한 대회용 리포지토리입니다.

- **Task**: Dialogue Summarization (Abstractive Summarization)  
- **Model**: `digit82/kobart-summarization`  
- **Goal**: 대화 내용을 **자연스럽게 한 문장으로 요약**  
- **Metric**: ROUGE-1 / ROUGE-2 / ROUGE-L (F1 평균)

본 리포지토리는  
**EDA → 전처리 전략 → KoBART Fine-Tuning → Ablation Study → 최종 제출**  
전체 과정을 재현할 수 있도록 구성되어 있습니다.

---

## 📂 프로젝트 구조

```bash
nlp-competition
│
├── configs/
│   └── config.yaml
│
├── data/
│   ├── raw/               # 대회 제공 원본 데이터
│   └── processed/         # 전처리 데이터 저장 공간
│
├── notebooks/
│   ├── 01_eda/
│   │   └── eda_v2_input_only.ipynb
│   ├── 02_preprocessing/
│   │   └── preprocessing_v2_input_only.ipynb
│   ├── 03_modeling/
│   │   └── modeling_kobart.ipynb
│   ├── baseline.ipynb
│   └── baseline_solar.ipynb
│
├── outputs/
│   ├── prediction/
│   │   ├── output_preprocessed_v2_input_only.csv
│   │   └── output_preprocessed_v2_input_only_wd0.01_ls0.1.csv
│   └── exp_log.csv
│
├── src/
│
├── .gitignore
└── README.md
````

---

## 🧪 대회 & 데이터 요약

### 🎯 대회 정보

* **대회 이름**: Dialogue Summarization
* **목표**: 일상 대화를 입력받아 핵심 요약문 1문장을 생성
* **평가 지표**: ROUGE-1 / ROUGE-2 / ROUGE-L F1 평균

### 🧾 데이터 구성

| 파일        | 개수     | 설명                         |
| --------- | ------ | -------------------------- |
| train.csv | 12,410 | dialogue + summary + topic |
| dev.csv   | 498    | 검증용                        |
| test.csv  | 499    | summary 없음                 |

각 샘플 예시:

```
fname,dialogue,summary,topic
train_0,"#Person1#: ...", "건강검진을 받으러 온 Mr. Smith...", 건강검진
```

---

## 🔍 EDA & 전처리 전략

### 1️⃣ EDA 핵심 인사이트

* Dialogue 길이 평균 **320–360 tokens**, Long-tail 존재
* Summary는 대부분 **한 문장, 100자 내외**
* topic은 summary에 직접 등장 비율은 낮지만 의미적 방향성은 제공
* “ㅋㅋ”, “음…”, “아…” 같은 noise 다수 → 제거 필요

### 2️⃣ 전처리 버전 실험 요약

| 버전                | 설명                              | 결과                       |
| ----------------- | ------------------------------- | ------------------------ |
| v1                | baseline 전처리                    | 참고용                      |
| v2_full           | 입력 + 출력 모두 전처리                  | 출력 손상 → 성능 저하            |
| **v2_input_only** | 입력만 공격적으로 정제                    | **성능 +0.5pt 향상 → 최종 채택** |
| v3–v6             | truncation, prefix, filtering 등 | 정보 손실/분포 왜곡 → 대부분 성능 하락  |

📌 **결론: "입력만 전처리" 전략이 최적. 출력(summary)은 절대 손대지 않는다.**

---

## 🤖 모델링 (KoBART Fine-Tuning)

### ⚙ 모델 설정

* **모델**: `digit82/kobart-summarization`
* **encoder_max_len**: 512
* **decoder_max_len**: 100

### 🧩 최종 학습 설정 (Best)

| 항목              | 값          |
| --------------- | ---------- |
| Epochs          | 20         |
| LR              | 1e-5       |
| Train batch     | 50         |
| Weight decay    | **0.01**   |
| Label smoothing | **0.1**    |
| Warmup          | 0.1        |
| Scheduler       | cosine     |
| Early stopping  | patience=3 |

### 💡 핵심 개선 요인

* **Label smoothing 0.1 → 요약 태스크에 특히 효과적**
* **Weight decay 0.01 → 일반화 성능 상승**
* **v2_input_only 전처리 → encoder representation 개선**

---

## 📊 성능 결과

| 실험명                              | 전처리      | 주요 설정            | Dev ROUGE-L | LB          |
| -------------------------------- | -------- | ---------------- | ----------- | ----------- |
| baseline_v1                      | baseline | 기본               | ~0.27       | 47.06       |
| v2_input_only                    | 입력만 정제   | 기본               | 0.282       | 47.6004     |
| **v2_input_only + WD + LS (최종)** | 입력만 정제   | LS 0.1 + WD 0.01 | **0.2978**  | **47.8581** |

---

## 🧭 전체 파이프라인

```
Raw CSV
    ↓
EDA 분석
    ↓
Preprocess_v2_input_only
    ↓
KoBART Fine-tuning
    ↓
Best checkpoint 평가
    ↓
Beam Search (4 beams)
    ↓
최종 제출 파일 생성
```

---

## 🛠 사용 방법 (How to Run)

### 1️⃣ 데이터 준비

```
data/raw/train.csv
data/raw/dev.csv
data/raw/test.csv
data/raw/sample_submission.csv
```

### 2️⃣ EDA 실행

→ `notebooks/01_eda/eda_v2_input_only.ipynb`

### 3️⃣ 전처리 실행

→ `notebooks/02_preprocessing/preprocessing_v2_input_only.ipynb`

### 4️⃣ 모델 학습

→ `notebooks/03_modeling/modeling_kobart.ipynb`

### 5️⃣ 제출 파일

```
outputs/prediction/output_preprocessed_v2_input_only_wd0.01_ls0.1.csv
```

---

## 🧠 회고 & 향후 개선 아이디어

### ✨ 배운 점

* 입력만 정제해도 모델 품질이 크게 향상됨
* Label smoothing은 요약 태스크에 매우 적합
* Decoding 튜닝보다 **모델/전처리 설계가 우선순위**

### 🔮 개선 아이디어

* QLoRA / PEFT
* Topic-aware prefix tuning
* 에러 분석 기반 후처리
* Prompt형 구조적 decoding

---

## 🙋‍♀️ 팀 & 브랜치 전략

| 브랜치             | 역할          |
| --------------- | ----------- |
| **main**        | 최종 안정 버전    |
| **dev**         | 핵심 실험/재현 파일 |

---

## 📫 Contact

문의는 GitHub Issue 또는 PR로 남겨주세요.

<!-- Header -->
<div align="center">

![header](https://capsule-render.vercel.app/api?type=rounded&height=170&text=Dialogue%20Summarization&desc=%EC%9D%BC%EC%83%81%20%EB%8C%80%ED%99%94%20%EC%9A%94%EC%95%BD%20NLP%20%EB%8C%80%ED%9A%8C%20%EC%84%A0%ED%98%95%20%EB%AA%A8%EB%8D%B8&fontSize=35&descSize=16&descAlignY=65&color=gradient&fontColor=ffffff&animation=fadeIn)

<h3>🗣️ KoBART 기반 일상 대화 요약 모델 · Dialogue Summarization Competition 🗣️</h3>

<!-- 원하면 repo 주소로 바꿔도 됨
[![GitHub stars](https://img.shields.io/github/stars/USER/REPO?style=social)](https://github.com/USER/REPO)
[![GitHub forks](https://img.shields.io/github/forks/USER/REPO?style=social)](https://github.com/USER/REPO)
-->

</div>

---

## 💻 프로젝트 소개

### 📌 프로젝트 개요

이 프로젝트는 **일상 대화(dialogue)를 입력으로 받아 한 문장 요약(summary)을 생성하는 NLP 모델**을 구축한 대회용 리포지토리입니다.

- **Task**: Dialogue Summarization (Abstractive Summarization)  
- **Model**: `digit82/kobart-summarization` (KoBART 기반 한국어 요약 모델)  
- **Goal**: 일상 대화 로그를 바탕으로, **핵심 내용을 1문장으로 자연스럽게 요약하는 모델** 만들기  
- **Metric**: ROUGE-1 / ROUGE-2 / ROUGE-L F1 평균  

대회에서 제공된 베이스라인 모델을 시작점으로,  
**EDA → 전처리 전략 → KoBART 파인튜닝 → 실험·Ablation**까지 전체 과정을 정리했습니다.

---

## 📂 프로젝트 구조

> 이 리포지토리는 **최종 결과 재현에 필요한 핵심 파일만 정리하여 업로드**되어 있습니다.

```bash
nlp-competition
│
├── configs/
│   └── config.yaml                    # 경로, 모델명, 학습/추론 설정
│
├── data/
│   ├── raw/                           # 대회 제공 원본 데이터 (train/dev/test) 위치
│   └── processed/                     # 전처리된 데이터 저장 경로
│
├── notebooks/
│   ├── 01_eda/
│   │   └── eda_v2_input_only.ipynb    # 데이터 EDA
│   ├── 02_preprocessing/
│   │   └── preprocessing_v2_input_only.ipynb   # 최종 전처리 파이프라인
│   ├── 03_modeling/
│   │   └── modeling_kobart.ipynb      # KoBART 학습/검증
│   ├── baseline.ipynb                 # 대회 베이스라인 재현
│   └── baseline_solar.ipynb           # Solar 기반 후처리/실험용 노트북
│
├── outputs/
│   ├── prediction/
│   │   ├── output_preprocessed_v2_input_only.csv
│   │   └── output_preprocessed_v2_input_only_wd0.01_ls0.1.csv  # 최종 제출 파일
│   └── exp_log.csv                    # 주요 실험 기록 (dev/LB 결과 요약)
│
├── src/                               # (선택) 추후 .py 코드로 정리할 공간
│
├── .gitignore
└── README.md

🧪 대회 & 데이터 요약
🎯 대회 정보

대회 이름: Dialogue Summarization | 일상 대화 요약

목표: 일상 대화를 입력받아, 대화의 핵심을 한 문장으로 요약하는 모델 구축

평가 지표:

ROUGE-1 F1

ROUGE-2 F1

ROUGE-L F1
→ 세 점수의 평균으로 최종 평가

🧾 데이터 구성

train.csv: 12,410개 (fname, dialogue, summary, topic)

dev.csv: 498개

test.csv: 499개 (summary 없음)

각 샘플은 아래와 같은 구조를 가짐:

fname,dialogue,summary,topic
train_0,"#Person1#: 안녕하세요, Mr. Smith. 저는 Dr. Hawkins입니다. 오늘 무슨 일로 오셨어요? 
#Person2#: 건강검진을 받으려고 왔어요.
...
#Person1#: 담배가 폐암하고 심장병의 주된 원인인 거 아시죠? 끊으셔야 해요.
#Person2#: 네, 고맙습니다, 의사 선생님.",
"Mr. Smith는 건강검진을 받으러 와서 매년 검진 필요성과 금연 관련 도움을 안내받는다.",건강검진

🔍 EDA & 전처리 전략
1️⃣ EDA 핵심 인사이트

Dialogue 길이

평균: 약 320–360 tokens

Long-tail 분포: 매우 긴 대화 다수 → truncation 시 정보 손실 위험

Summary 길이

평균: 100–110자

거의 모두 1문장 요약, 문체도 일정 (“~다.”, “~했다.”)

Topic

topic 문자열이 summary에 직접 등장하는 비율은 낮음(10–20%)

다만 의미적으로는 summary와 높은 관련 → “방향성” 정도의 역할

Noise

ㅋㅋㅋ, ㅎㅎ, ㅠㅠ, 아…, 음… 등 감정/잡담성 토큰 다수

summary에는 거의 반영되지 않음 → 공격적인 입력 전처리 후보

2️⃣ 전처리 버전 실험
버전	내용	결과
v1	baseline 전처리	기준 점수
v2_full	입력 + 출력 모두 전처리	출력까지 건드리면 label bias 발생, 효과 없음
v2_input_only	입력만 공격적으로 정제	LB +0.5pt 수준 향상 (채택)
v3–v6	truncation, topic prefix, 길이/overlap filtering 등	데이터 분포 왜곡·정보 손실로 대부분 성능 저하

📌 결론
→ 입력(dialogue)은 강하게 정제, 출력(summary)은 최대한 건드리지 않는 전략이 최선
→ 이 프로젝트의 최종 전처리 파이프라인은 Preprocess_v2_input_only 입니다.

🤖 모델링 (KoBART Fine-tuning)
⚙ 사용 모델

digit82/kobart-summarization

구조: Encoder–Decoder (BART 계열)

토크나이저:

encoder_max_len = 512

decoder_max_len = 100

특수 토큰 등록: #Person1#, #PhoneNumber#, #Address# 등

🧩 학습 설정 (최종 Best 설정)
항목	값
Epochs	20
Learning rate	1e-5
Train batch size	50
Eval batch size	32
Weight decay	0.01
Label smoothing	0.1
Warmup ratio	0.1
Scheduler	cosine
Optimizer	AdamW (adamw_torch)
Early stopping	patience = 3 (metric: eval_rouge_l)
Seed	42
💡 핵심 튜닝 포인트

Label smoothing = 0.1

요약은 “정답이 여러 표현으로 존재”하는 태스크 → overconfidence 방지에 효과적

dev & LB 모두에서 가장 안정적인 향상

Weight decay = 0.01

KoBART 파라미터의 과적합을 억제해 일반화 성능 개선

입력 전처리 v2_input_only

노이즈 제거 + topic 누출 방지 → encoder representation 품질 향상

📊 성능 결과
✅ 주요 실험 비교
실험명	전처리	주요 설정	Dev ROUGE-L	LB 최종 점수
baseline_v1	baseline	기본 설정	~0.27대	47.06
v2_input_only	입력만 정제	기본 설정	0.282	47.6004
v2_input_only + WD 0.01 + LS 0.1	입력만 정제	LS=0.1, WD=0.01	0.2978	47.8581 (최종)

📌 정리
→ 성능 향상에 가장 크게 기여한 요소는

입력 전처리(v2_input_only)

Label smoothing 0.1

Weight decay 0.01 이었습니다.

🧭 전체 파이프라인
Raw CSV (train/dev/test)
        ↓
        ↓  [01_eda/eda_v2_input_only.ipynb]
    EDA 분석
        ↓
        ↓  [02_preprocessing/preprocessing_v2_input_only.ipynb]
Preprocess_v2_input_only
  (noise 제거 + 누출 방지)
        ↓
        ↓  [03_modeling/modeling_kobart.ipynb]
KoBART Fine-tuning
  (LS 0.1, WD 0.01, 20 epochs)
        ↓
Best checkpoint 선택 (eval_rouge_l 기준)
        ↓
Beam Search (num_beams=4, max_len=100)
        ↓
outputs/prediction/output_preprocessed_v2_input_only_wd0.01_ls0.1.csv

🛠 사용 방법 (How to Run)

⚠️ 대회 데이터는 공개 저장소에 포함되어 있지 않으므로,
반드시 개별적으로 다운로드 후 data/raw/ 디렉토리에 위치시켜야 합니다.

1️⃣ 데이터 준비
data/raw/train.csv
data/raw/dev.csv
data/raw/test.csv
data/raw/sample_submission.csv

2️⃣ EDA & 전처리

EDA:
👉 notebooks/01_eda/eda_v2_input_only.ipynb

전처리 파이프라인:
👉 notebooks/02_preprocessing/preprocessing_v2_input_only.ipynb

3️⃣ 학습 & 평가

KoBART 학습/검증:
👉 notebooks/03_modeling/modeling_kobart.ipynb

노트북에서 config.yaml을 기반으로 경로/하이퍼파라미터를 로드하여 학습을 재현할 수 있습니다.

4️⃣ 추론 & 제출 파일 생성

최종 제출 파일:

outputs/prediction/output_preprocessed_v2_input_only_wd0.01_ls0.1.csv

Baseline과 비교용:

outputs/prediction/output_preprocessed_v2_input_only.csv

🧠 회고 & 향후 개선 아이디어
✨ 이번 실험에서 배운 점

입력 전처리의 힘

감정/잡담성 토큰 제거만으로도 모델이 “핵심 발화”에 더 집중하게 됨.

Label smoothing은 요약 태스크에 특히 잘 맞는다

정답 문장이 하나로 고정되지 않는 태스크일수록,
overconfidence를 낮춰주는 regularization이 효과적.

Decoding 튜닝보다 모델/전처리 설계가 우선순위

length penalty, repetition penalty 등을 크게 바꿔도
모델 표현력이 충분하지 않으면 성능 상한을 넘기 어려움.

🔮 향후 개선 아이디어

QLoRA / PEFT 기반 파라미터 효율적 튜닝 시도

Topic 정보를 더 정교하게 활용하는 컨디셔닝 전략 (prefix tuning 등)

“한 문장 요약” 제약을 명시적으로 반영하는 decoding 규칙 설계

Hard case (실수 많은 샘플) 중심 에러 분석 + 규칙 기반 후처리 강화

🙋‍♀️ 팀 & 브랜치 전략

이 리포지토리는 다음과 같은 브랜치 전략으로 관리됩니다.

main : 최종 정리된 코드 & 결과만 반영하는 안정 브랜치

dev : 실험 재현을 위한 핵심 파일 (EDA/전처리/모델링 노트북, config 등)

yekyung-dev : 개인 실험, 로그, 실험용 코드(초안) 저장용 브랜치

협업 과정에서 나온 모든 실험은 yekyung-dev → dev → main 순으로 정리·압축되었습니다.

📫 Contact

프로젝트 관련 문의 또는 피드백은
👉 GitHub Issue 또는 Pull Request로 남겨 주세요.

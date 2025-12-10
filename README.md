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

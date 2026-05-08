# GCP Vertex AI TPM 할당량 소모 방식 검증

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/cjk0604/tpm-calculation/blob/main/tpm_quota_verification_handson.ipynb)
[![Open In Colab Enterprise](https://img.shields.io/badge/Colab_Enterprise-Open-blue?logo=google-cloud)](https://console.cloud.google.com/vertex-ai/colab/import/https:%2F%2Fraw.githubusercontent.com%2Fcjk0604%2Ftpm-calculation%2Fmain%2Ftpm_quota_verification_handson.ipynb)
[![View on GitHub](https://img.shields.io/badge/GitHub-View%20Notebook-181717?logo=github)](https://github.com/cjk0604/tpm-calculation/blob/main/tpm_quota_verification_handson.ipynb)

GCP Vertex AI 환경에서 TPM(Tokens Per Minute) 할당량이 실제로 어떻게 차감되는지 검증하는 핸즈온 테스트입니다.

**핵심 질문:** 캐시된 토큰(Cached Tokens)이 TPM 할당량에서 할인 없이 1:1로 차감되는가?

## 배경

이 테스트를 통해 확인한 `usageMetadata`의 토큰 카운트 구조는, 사내 개발자용 게이트웨이 프록시(LiteLLM 등)를 Cloud Run에 배포할 때 할당량 제한 로직과 BigQuery 토큰 사용량 로그 적재의 핵심 기준이 됩니다.

## 테스트 흐름

| 단계 | 내용 | 방법 |
|------|------|------|
| **1단계** | 12k 캐시 생성 + Gemini 2.5 Flash 10회 연속 호출 | Jupyter Notebook 실행 |
| **2단계** | GCP 콘솔 할당량 그래프에서 TPM 차감 확인 | 콘솔 모니터링 |

## 사전 요구사항

- GCP 프로젝트 (Vertex AI API 활성화)
- Python 3.9+
- `us-central1` 리전에서 Gemini 2.5 Flash 모델 접근 권한
- GCP 인증 완료 (`gcloud auth application-default login`)

## 빠른 시작

```bash
# 1. 저장소 클론
git clone https://github.com/cjk0604/tpm-calculation.git && cd tpm-calculation

# 2. 가상 환경 생성 및 패키지 설치
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 3. GCP 인증
gcloud auth application-default login

# 4. Jupyter 실행
jupyter notebook tpm_quota_verification_handson.ipynb
```

> **중요:** 노트북 실행 전 `PROJECT_ID`를 본인 GCP 프로젝트 ID로 변경하세요.

## 판정 기준

코드 실행 후 GCP 콘솔 (**IAM 및 관리자 > 할당량 및 시스템 한도**)에서:

- **그래프 피크 ≈ `prompt_token_count` 합계** → 캐시 토큰도 1:1로 TPM 차감
- **그래프 피크 ≈ `(prompt - cached)` 합계** → 캐시 토큰은 TPM에서 할인

## 파일 구조

```
tpm-calcuation/
├── tpm_quota_verification_handson.ipynb   # 메인 테스트 노트북
├── requirements.txt                       # Python 의존성
├── .gitignore
└── README.md
```

## 테스트 환경

- **모델:** `gemini-2.5-flash-preview-05-20`
- **리전:** `us-central1`
- **캐시 크기:** ~12,000 토큰 (시스템 프롬프트)
- **호출 횟수:** 10회 연속

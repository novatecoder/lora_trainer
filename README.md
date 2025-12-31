# 🚀 Lora-trainer

**Lora-trainer**는 서포터 AI(Supporter AI)의 핵심인 '혈액형별 페르소나'를 구현하기 위해 **Qwen2.5-7B-Instruct** 모델을 LoRA(Low-Rank Adaptation) 방식으로 미세 조정(Fine-tuning)하는 전용 프로젝트입니다. `Unsloth` 라이브러리를 사용하여 메모리 효율을 극대화하고 학습 속도를 높였습니다.

## 👤 Author

* **Kim Wonhyeok** [nnovatecoder@gmail.com](mailto:nnovatecoder@gmail.com)

## ✨ Key Features

* **Unsloth 기반 최적화**: 4-bit QLoRA를 통해 일반적인 HuggingFace 학습 대비 메모리 사용량 60% 절감 및 속도 2배 향상.
* **혈액형별 페르소나 학습**: A(다정), B(솔직), O(활기), AB(엉뚱) 등 혈액형별 고유의 말투와 리액션 패턴 구축.
* **티키타카(Tiki-Taka) 모드**: 혼자 길게 말하지 않고 짧은 문장(1~2문장)으로 대화를 주고받으며 사용자에게 질문을 던지는 능동적 대화 스타일 학습.
* **구조화된 출력(JSON)**: 서비스 노드와의 연동을 위해 항상 일정한 JSON 형식(`text`, `emotion`)으로 응답하도록 강제.
* **감정 지능 반영**: AI 내부의 감정 수치(`Intensity`)에 따라 발화 톤과 이모지 사용 빈도가 동적으로 변하는 학습 데이터 구조.

## 🛠 Tech Stack

* **Base Model**: [Qwen/Qwen2.5-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-7B-Instruct)
* **Training Library**: [Unsloth](https://github.com/unslothai/unsloth)
* **Frameworks**: PyTorch, HuggingFace PEFT & TRL
* **Environment**: Python 3.10+, Poetry

## 📂 Project Structure

```text
Lora-trainer/
├── data/               # 혈액형별 학습용 JSONL 데이터셋
├── outputs/            # 학습 완료된 LoRA 어댑터 가중치 (adapter_A, adapter_B 등)
├── scripts/
│   ├── train.py        # 메인 LoRA 학습 스크립트
│   └── generate_data.py # 티키타카 데이터셋 생성 도구
├── utils/
│   └── formatter.py    # ChatML 포맷팅 및 프롬프트 최적화 유틸
├── pyproject.toml      # 의존성 관리
└── README.md

```

## 🚀 Quick Start

### 1. 의존성 설치

`Poetry`를 사용하여 최적화된 학습 환경을 구축합니다.

```bash
poetry install

```

### 2. 데이터 준비

`data/` 폴더에 혈액형별 학습 데이터를 위치시킵니다. (형식: `train_data_type_A.jsonl`)

### 3. 학습 실행

특정 혈액형에 대한 LoRA 학습을 시작합니다.

```bash
poetry run python scripts/train.py --blood_type A

```

## 📝 Training Data Format

`expression_node`와 동일한 시스템 프롬프트 구조를 사용하여 학습합니다.

```json
{
  "instruction": "너는 A형 다정한 친구야. 짧은 반말로 답해.",
  "context": {
    "summary": "우리는 카페에서 만나기로 했어.",
    "current_emotion": {"type": "happy", "intensity": 0.8}
  },
  "input": "나 지금 도착했는데 어디야?",
  "output": "{\"text\": \"오! 나 금방 도착해! 1층 창가 자리에 앉아있어줘!\", \"emotion\": \"smile\"}"
}

```

## 🗺 Roadmap

상세한 개발 계획은 [ROADMAP.md](https://www.google.com/search?q=./ROADMAP.md)에서 확인할 수 있습니다.

## 📐 Design Specification

데이터 구조 및 모델 파라미터 규격은 [SDS.md](https://www.google.com/search?q=./SDS.md)를 참조하십시오.
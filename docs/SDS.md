# [SDS] Lora-trainer 상세 설계서

## 1. 시스템 개요 (System Overview)

본 프로젝트는 `supporter_ai` 서비스의 **Expression Node**에서 사용할 혈액형별 페르소나 어댑터를 생성하는 전용 학습 파이프라인이다. **Unsloth** 라이브러리를 활용하여 저사양 환경에서도 고성능의 4-bit QLoRA 학습을 수행하며, 최종적으로 혈액형별(A, B, O, AB) 독립된 LoRA 가중치 파일을 생성하는 것을 목적으로 한다.

## 2. 모델 규격 (Model Specification)

### 2.1 베이스 모델

* **Model**: `Qwen/Qwen2.5-7B-Instruct`
* **Reason**: 한국어 이해도가 높고, 지시 이행 능력이 뛰어나며 Unsloth 커널 최적화가 잘 되어 있음.

### 2.2 학습 기법 (Training Technique)

* **Method**: 4-bit QLoRA (via Unsloth)
* **Rank (R)**: 16 (표현의 다양성과 효율성 간의 균형)
* **Alpha**: 16
* **Target Modules**: `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj`
* **Precision**: `Mixed Precision (FP16/BF16)`

## 3. 데이터 및 프롬프트 설계 (Data & Prompt Design)

### 3.1 통합 시스템 프롬프트 템플릿 (Unified Template)

학습 시 사용되는 시스템 프롬프트의 구조는 실제 서비스(`expression_node`)와 100% 일치하도록 구성한다.

```text
<|im_start|>system
너는 {blood_type}형 {persona} 친구야. 
- 오직 한국어 반말만 사용. 중국어 절대 금지.
- 기억: {summary}
- 규칙: 짧게 한두 문장으로만 말해. 혼자 길게 떠들지 말고 질문을 던지거나 리액션만 해. 대화를 이어가는 게 목적이야.
- 형식: {{ "text": "할말", "emotion": "표정" }}<|im_end|>

```

### 3.2 데이터셋 구조 (JSONL)

각 혈액형별로 독립된 데이터셋을 구성하며, 다음 필드를 포함한다.

| 필드명 | 설명 | 비고 |
| --- | --- | --- |
| `instruction` | 혈액형 및 페르소나 지시문 | 시스템 프롬프트 영역 |
| `context` | `summary` 및 `emotion_state` | 대화 맥락 정보 |
| `input` | 사용자의 최신 발화 | `input_text` |
| `output` | AI의 최종 답변 (JSON) | `{"text": "...", "emotion": "..."}` |

## 4. 학습 파이프라인 (Training Pipeline)

### 4.1 데이터 생성 (Data Generation)

* **전략**: 티키타카(Tiki-Taka)형 대화 생성.
* **특징**: 단순히 정보를 전달하는 것이 아니라, 상대방의 말을 받아치고 질문을 던지는 상호작용 위주의 데이터를 구성.
* **감정 반영**: AI의 내부 감정 상태에 따라 문장 끝 어미나 이모지 사용 빈도를 다르게 학습.

### 4.2 학습 파라미터 (Hyperparameters)

* **Batch Size**: 2 (GPU 메모리 상황에 따라 조절)
* **Gradient Accumulation**: 4
* **Learning Rate**: 2e-4
* **Epochs/Steps**: 데이터셋 크기에 따라 1~3 Epoch 수행.
* **Max Sequence Length**: 2048

## 5. 출력 및 검증 규격 (Output & Evaluation)

### 5.1 출력물 (Artifacts)

* **LoRA 어댑터**: `adapter_A`, `adapter_B`, `adapter_O`, `adapter_AB` 폴더별 저장.
* **설정 파일**: `adapter_config.json`, `adapter_model.bin`.

### 5.2 검증 지표

* **JSON 준수율**: 생성된 결과가 정해진 JSON 형식을 100% 유지하는가.
* **한국어 전용성**: 답변 중 중국어 한자가 포함되지 않는가.
* **페르소나 일치도**: A형(다정), B형(솔직) 등 각 혈액형별 말투가 뚜렷하게 구분되는가.

## 6. 보안 및 예외 처리

* **중국어 감지**: 학습 중 중국어 환각이 발생할 경우 `safe_llm_call` 로직에 따라 재시도하거나 필터링하여 데이터 품질 유지.
* **데이터 개인정보**: 학습 데이터 생성 시 실제 개인정보가 포함되지 않도록 가공 처리.

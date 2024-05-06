# LLM 파인튜닝

AutoTrain을 사용하면 자신의 데이터에서 대형 언어 모델(LLM)을 쉽게 파인튜닝할 수 있습니다!

AutoTrain은 다음과 같은 유형의 LLM 파인튜닝을 지원합니다:

- Causal Language Modeling (CLM)
- Masked Language Modeling (MLM) [곧 출시 예정]

## 데이터 준비

LLM 파인튜닝은 CSV 형식의 데이터를 받습니다.

### SFT / Generic Trainer를 위한 데이터 형식

SFT / Generic Trainer의 경우 데이터는 다음 형식이어야 합니다:

| text                                                          |
| ------------------------------------------------------------- |
| human: hello \n bot: hi nice to meet you                      |
| human: how are you \n bot: I am fine                          |
| human: What is your name? \n bot: My name is Mary             |
| human: Which is the best programming language? \n bot: Python |

이 형식의 예제 데이터셋은 여기서 찾을 수 있습니다: https://huggingface.co/datasets/timdettmers/openassistant-guanaco

SFT/Generic 훈련을 위해서는 데이터셋에 `text` 열이 있어야 합니다.

### Reward Trainer를 위한 데이터 형식

Reward Trainer의 경우 데이터는 다음 형식이어야 합니다:

| text                                                          | rejected_text                                                     |
| ------------------------------------------------------------- | ----------------------------------------------------------------- |
| human: hello \n bot: hi nice to meet you                      | human: hello \n bot: leave me alone                               |
| human: how are you \n bot: I am fine                          | human: how are you \n bot: I am not fine                          |
| human: What is your name? \n bot: My name is Mary             | human: What is your name? \n bot: Whats it to you?                |
| human: Which is the best programming language? \n bot: Python | human: Which is the best programming language? \n bot: Javascript |

Reward Trainer의 경우 데이터셋에 `text` 열(선택된 텍스트)과 `rejected_text` 열이 있어야 합니다.

### DPO Trainer를 위한 데이터 형식

DPO Trainer의 경우 데이터는 다음 형식이어야 합니다:

| prompt                                  | text                | rejected_text      |
| --------------------------------------- | ------------------- | ------------------ |
| hello                                   | hi nice to meet you | leave me alone     |
| how are you                             | I am fine           | I am not fine      |
| What is your name?                      | My name is Mary     | Whats it to you?   |
| What is your name?                      | My name is Mary     | I dont have a name |
| Which is the best programming language? | Python              | Javascript         |
| Which is the best programming language? | Python              | C++                |
| Which is the best programming language? | Java                | C++                |

DPO Trainer의 경우 데이터셋에 `prompt` 열, `text` 열(선택된 텍스트), 그리고 `rejected_text` 열이 있어야 합니다.

모든 작업에 대해 CSV와 JSONL 파일을 모두 사용할 수 있습니다!

## 매개변수

```
❯ autotrain llm --help
usage: autotrain <command> [<args>] llm [-h] [--train] [--deploy] [--inference] [--username USERNAME]
                                        [--backend {local-cli,spaces-a10gl,spaces-a10gs,spaces-a100,spaces-t4m,spaces-t4s,spaces-cpu,spaces-cpuf}]
                                        [--token TOKEN] [--push-to-hub] --model MODEL --project-name PROJECT_NAME [--data-path DATA_PATH]
                                        [--train-split TRAIN_SPLIT] [--valid-split VALID_SPLIT] [--batch-size BATCH_SIZE] [--seed SEED]
                                        [--epochs EPOCHS] [--gradient_accumulation GRADIENT_ACCUMULATION] [--disable_gradient_checkpointing]
                                        [--lr LR] [--log {none,wandb,tensorboard}] [--text_column TEXT_COLUMN]
                                        [--rejected_text_column REJECTED_TEXT_COLUMN] [--prompt-text-column PROMPT_TEXT_COLUMN]
                                        [--model-ref MODEL_REF] [--warmup_ratio WARMUP_RATIO] [--optimizer OPTIMIZER] [--scheduler SCHEDULER]
                                        [--weight_decay WEIGHT_DECAY] [--max_grad_norm MAX_GRAD_NORM] [--add_eos_token] [--block_size BLOCK_SIZE]
                                        [--peft] [--lora_r LORA_R] [--lora_alpha LORA_ALPHA] [--lora_dropout LORA_DROPOUT]
                                        [--logging_steps LOGGING_STEPS] [--evaluation_strategy {epoch,steps,no}]
                                        [--save_total_limit SAVE_TOTAL_LIMIT] [--auto_find_batch_size]
                                        [--mixed_precision {fp16,bf16,None}] [--quantization {int4,int8,None}] [--model_max_length MODEL_MAX_LENGTH]
                                        [--max_prompt_length MAX_PROMPT_LENGTH] [--max_completion_length MAX_COMPLETION_LENGTH]
                                        [--trainer {default,dpo,sft,orpo,reward}] [--target_modules TARGET_MODULES] [--merge_adapter]
                                        [--use_flash_attention_2] [--dpo-beta DPO_BETA] [--chat_template {tokenizer,chatml,zephyr,None}]
                                        [--padding {left,right,None}]

✨ AutoTrain LLM 실행

options:
  -h, --help            도움말 메시지를 표시하고 종료합니다
  --train               모델 훈련 명령
  --deploy              모델 배포 명령 (제한된 사용 가능)
  --inference           추론 실행 명령 (제한된 사용 가능)
  --username USERNAME   Hugging Face Hub 사용자 이름
  --backend {local-cli,spaces-a10gl,spaces-a10gs,spaces-a100,spaces-t4m,spaces-t4s,spaces-cpu,spaces-cpuf}
                        사용할 백엔드: 기본값 또는 공간. Spaces 백엔드는 push_to_hub 및 username이 필요합니다. 고급 사용자 전용.
  --token TOKEN         Hugging Face API 토큰. 토큰은 모델 허브에 대한 쓰기 권한이 있어야 합니다.
  --push-to-hub         훈련 후 허브로 푸시하면 훈련된 모델이 Hugging Face 모델 허브로 푸시됩니다.
  --model MODEL         훈련에 사용할 기본 모델
  --project-name PROJECT_NAME
                        훈련된 모델에 대한 출력 디렉터리/리포지토리 ID (허브에서 고유해야 함)
  --data-path DATA_PATH
                        사용할 훈련 데이터셋. cli를 사용할 때는 적절한 형식으로 훈련 및 검증 데이터가 포함된 디렉터리 경로여야 합니다
  --train-split TRAIN_SPLIT
                        사용할 훈련 데이터셋 분할
  --valid-split VALID_SPLIT
                        사용할 검증 데이터셋 분할
  --batch-size BATCH_SIZE, --train-batch-size BATCH_SIZE
                        사용할 훈련 배치 크기
  --seed SEED           재현성을 위한 랜덤 시드
  --epochs EPOCHS       훈련 에포크 수
  --gradient_accumulation GRADIENT_ACCUMULATION, --gradient-accumulation GRADIENT_ACCUMULATION
                        그래디언트 누적 단계
  --disable_gradient_checkpointing, --disable-gradient-checkpointing, --disable-gc
                        그래디언트 체크포인트를 비활성화합니다
  --lr LR               학습률
  --log {none,wandb,tensorboard}
                        실험 추적 사용
  --text_column TEXT_COLUMN, --text-column TEXT_COLUMN
                        텍스트 데이터에 사용할 데이터셋 열을 지정합니다. 이 매개변수는 텍스트 정보를 처리하는 모델에 필수적입니다. 기본값은 'text'입니다.
  --rejected_text_column REJECTED_TEXT_COLUMN, --rejected-text-column REJECTED_TEXT_COLUMN
                        처리하기에 적합하지 않은 기준을 만족하지 않는 항목을 저장하는 데 사용할 열을 정의합니다. 기본값은 'rejected'입니다.
                        orpo, dpo 및 reward trainer에서만 사용됩니다.
  --prompt-text-column PROMPT_TEXT_COLUMN, --prompt-text-column PROMPT_TEXT_COLUMN
                        대화 또는 완성 생성과 같은 맥락적 입력이 필요한 작업을 위한 프롬프트 텍스트가 포함된 열을 식별합니다. 기본값은 'prompt'입니다.
                        dpo trainer에서만 사용됩니다.
  --model-ref MODEL_REF
                        PEFT를 사용하지 않을 때 DPO에 사용할 참조 모델
  --warmup_ratio WARMUP_RATIO, --warmup-ratio WARMUP_RATIO
                        학습률을 워밍업하는 데 할당된 훈련 비율을 설정합니다. 이는 훈련 시작 시 모델의 안정성과 성능을 높일 수 있습니다.
                        기본값은 0.1입니다.
  --optimizer OPTIMIZER
                        모델을 훈련시키기 위한 최적화 알고리즘을 선택합니다. 다른 최적화기는 훈련 속도와 모델 성능에 영향을 줄 수 있습니다.
                        기본적으로 'adamw_torch'가 사용됩니다.
  --scheduler SCHEDULER
                        에포크 수를 기준으로 학습률을 조정할 학습률 스케줄러를 선택합니다. 'linear'는 초기 설정된 lr에서 선형으로 학습률을 감소시킵니다.
                        기본값은 'linear'입니다. 코사인 어닐링 스케줄을 위해 'cosine'을 시도해 보세요.
  --weight_decay WEIGHT_DECAY, --weight-decay WEIGHT_DECAY
                        과적합을 방지하기 위해 더 큰 가중치에 페널티를 주는 데 도움이 되는 정규화를 위한 가중치 감쇠율을 정의합니다.
                        기본값은 0.0입니다.
  --max_grad_norm MAX_GRAD_NORM, --max-grad-norm MAX_GRAD_NORM
                        역전파 중에 그래디언트 폭발을 방지하는 데 중요한 그래디언트 클리핑에 대한 최대 노름을 설정합니다.
                        기본값은 1.0입니다.
  --add_eos_token, --add-eos-token
                        언어 모델과 같은 특정 유형의 모델에 중요할 수 있는 텍스트 끝에 자동으로 문장 끝(EOS) 토큰을 추가할지 여부를 전환합니다.
                        `default` 트레이너에서만 사용됩니다.
  --block_size BLOCK_SIZE, --block-size BLOCK_SIZE
                        시퀀스를 처리하기 위한 블록 크기를 지정합니다. 이는 최대 시퀀스 길이 또는 하나의 텍스트 블록의 길이입니다. -1로 설정하면 블록 크기가 자동으로 결정됩니다.
                        기본값은 -1입니다.
  --peft, --use-peft    LoRA-PEFT 활성화
  --lora_r LORA_R, --lora-r LORA_R
                        Low-Rank Adaptation (LoRA)의 'r' 매개변수를 설정합니다. 기본값은 16입니다.
  --lora_alpha LORA_ALPHA, --lora-alpha LORA_ALPHA
                        LoRA의 'alpha' 매개변수를 지정합니다. 기본값은 32입니다.
  --lora_dropout LORA_DROPOUT, --lora-dropout LORA_DROPOUT
                        적응 중 과적합을 방지하는 데 도움이 되도록 LoRA 계층 내의 드롭아웃 비율을 설정합니다. 기본값은 0.05입니다.
  --logging_steps LOGGING_STEPS, --logging-steps LOGGING_STEPS
                        단계별로 훈련 진행 상황을 얼마나 자주 로깅할지 결정합니다. '-1'로 설정하면 로깅 단계가 자동으로 결정됩니다.
  --evaluation_strategy {epoch,steps,no}, --evaluation-strategy {epoch,steps,no}
                        모델의 성능을 평가하는 빈도를 선택하며, 'epoch'가 기본값으로 각 훈련 에포크 끝에 평가합니다.
  --save_total_limit SAVE_TOTAL_LIMIT, --save-total-limit SAVE_TOTAL_LIMIT
                        디스크 사용량을 효과적으로 관리하기 위해 저장된 총 모델 체크포인트 수를 제한합니다. 기본값은 최신 체크포인트만 저장하는 것입니다.
  --auto_find_batch_size, --auto-find-batch-size
                        시스템 성능을 기반으로 효율성을 최대화하기 위해 최적의 배치 크기를 자동으로 결정합니다.
  --mixed_precision {fp16,bf16,None}, --mixed-precision {fp16,bf16,None}
                        성능과 메모리 사용량을 최적화하기 위해 훈련에 사용할 정밀도 모드를 선택합니다. 옵션은 'fp16', 'bf16' 또는 기본 정밀도인 None입니다.
                        기본값은 None입니다.
  --quantization {int4,int8,None}, --quantization {int4,int8,None}
                        모델 크기를 줄이고 잠재적으로 추론 속도를 높이기 위해 양자화 수준을 선택합니다. 옵션에는 'int4', 'int8' 또는 None이 포함됩니다.
                        활성화하려면 --peft가 필요합니다.
  --model_max_length MODEL_MAX_LENGTH, --model-max-length MODEL_MAX_LENGTH
                        단일 배치에서 처리할 모델의 최대 길이를 설정하며, 이는 성능과 메모리 사용량에 모두 영향을 줄 수 있습니다.
                        기본값은 1024입니다.
  --max_prompt_length MAX_PROMPT_LENGTH, --max-prompt-length MAX_PROMPT_LENGTH
                        초기 맥락 입력이 필요한 작업에 특히 관련이 있는 훈련에 사용되는 프롬프트의 최대 길이를 지정합니다.
                        `orpo` 트레이너에서만 사용됩니다.
  --max_completion_length MAX_COMPLETION_LENGTH, --max-completion-length MAX_COMPLETION_LENGTH
                        사용할 완성 길이입니다. orpo의 경우 인코더-디코더 모델에서만 사용합니다.
  --trainer {default,dpo,sft,orpo,reward}
                        사용할 트레이너 유형
  --target_modules TARGET_MODULES, --target-modules TARGET_MODULES
                        LoRA와 같은 적응 또는 최적화를 위해 모델 아키텍처 내에서 대상으로 할 특정 모듈을 식별합니다.
                        쉼표로 구분된 모듈 이름 목록입니다. 기본값은 'all-linear'입니다.
  --merge_adapter, --merge-adapter
                        PEFT 어댑터를 모델과 병합하려면 이 플래그를 사용하세요
  --use_flash_attention_2, --use-flash-attention-2, --use-fa2
                        flash attention 2 사용
  --dpo-beta DPO_BETA, --dpo-beta DPO_BETA
                        DPO 트레이너의 베타값
  --chat_template {tokenizer,chatml,zephyr,None}, --chat-template {tokenizer,chatml,zephyr,None}
                        'tokenizer', 'chatml', 'zephyr' 또는 None을 포함한 옵션으로 채팅 기반 상호 작용을 위한 특정 템플릿을 적용합니다.
                        이 설정은 모델의 대화 행동을 형성할 수 있습니다.
  --padding {left,right,None}, --padding {left,right,None}
                        입력 정렬에 민감한 모델에 중요한 시퀀스의 패딩 방향을 지정합니다. 옵션에는 'left', 'right' 또는 None이 포함됩니다.
```

AutoTrain은 LLM 파인튜닝을 위한 다양한 옵션과 매개변수를 제공합니다. 이를 통해 사용자는 자신의 특정 요구 사항에 맞게 훈련 프로세스를 맞춤 설정할 수 있습니다.

데이터 준비 측면에서 AutoTrain은 다양한 훈련 작업(예: SFT, Reward 트레이너, DPO 트레이너)에 대해 특정 데이터 형식을 요구합니다. 데이터는 CSV 또는 JSONL 형식으로 제공될 수 있습니다.

훈련 매개변수와 관련하여 AutoTrain은 배치 크기, 에포크 수, 학습률, 최적화기, 스케줄러 등을 포함한 다양한 옵션을 제공합니다. 이러한 매개변수는 모델 성능과 훈련 효율성에 상당한 영향을 미칠 수 있습니다.

AutoTrain은 또한 LoRA(Low-Rank Adaptation)와 같은 고급 기술을 통합하여 계산 및 메모리 효율성을 개선하는 동시에 강력한 성능을 달성할 수 있습니다.

전반적으로 AutoTrain은 사용자가 최소한의 노력으로 LLM을 파인튜닝할 수 있는 포괄적이고 사용하기 쉬운 프레임워크를 제공합니다. 다양한 옵션과 매개변수를 통해 사용자는 자신의 특정 사용 사례에 맞게 훈련 프로세스를 최적화할 수 있습니다.

AutoTrain을 사용하면 머신러닝 전문 지식이 거의 또는 전혀 없는 사용자도 최첨단 LLM을 훈련시키고 강력한 AI 애플리케이션을 구축할 수 있습니다. 이는 LLM의 힘을 보다 광범위한 사용자와 사용 사례에 제공하는 데 도움이 될 수 있습니다.

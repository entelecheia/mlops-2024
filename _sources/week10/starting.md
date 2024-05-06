# 시작하기

## CLI 시작하기

autotrain cli를 로컬이나 Colab에서 실행하려면, autotrain-advanced 파이썬 패키지를 설치하세요:

```bash
pip install autotrain-advanced
```

그리고 다음 명령을 실행하세요:

```bash
export HF_TOKEN=your_hugging_face_write_token
autotrain --help
```

이렇게 하면 앱이 `http://127.0.0.1:8000`에서 시작됩니다.

AutoTrain은 pytorch, torchaudio, torchvision 또는 다른 종속성을 자동으로 설치하지 않습니다. 따라서 별도로 설치해야 합니다.
그래서 conda 환경을 사용하는 것이 좋습니다:

```bash
conda create -n autotrain python=3.10
conda activate autotrain

pip install autotrain-advanced

conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
conda install -c "nvidia/label/cuda-12.1.0" cuda-nvcc
conda install xformers -c xformers

python -m nltk.downloader punkt
pip install flash-attn --no-build-isolation
pip install deepspeed

export HF_TOKEN=your_hugging_face_write_token
autotrain --help
```

이렇게 하면 사용 가능한 CLI 명령어가 표시됩니다:

```bash
autotrain --help
```

```bash
usage: autotrain <command> [<args>]

positional arguments:
{
    app,
    llm,
    setup,
    dreambooth,
    api,
    text-classification,
    image-classification,
    tabular,
    spacerunner,
    seq2seq,
    token-classification
}

    commands

options:
  -h, --help            show this help message and exit
  --version, -v         Display AutoTrain version

For more information about a command, run: `autotrain <command> --help`
```

최종 사용자가 관심 있어 할 autotrain 명령어는 다음과 같습니다:

- `app`: AutoTrain UI 시작
- `llm`: 언어 모델 훈련
- `dreambooth`: DreamBooth를 사용하여 모델 훈련
- `text-classification`: 텍스트 분류 모델 훈련
- `image-classification`: 이미지 분류 모델 훈련
- `tabular`: 테이블 데이터 모델 훈련
- `spacerunner`: SpaceRunner를 사용하여 사용자 지정 모델 훈련
- `seq2seq`: 시퀀스-투-시퀀스 모델 훈련
- `token-classification`: 토큰 분류 모델 훈련

문제가 있는 경우 [GitHub issues](https://github.com/huggingface/autotrain-advanced/)에 보고해 주세요.

## UI 시작하기

AutoTrain UI는 필요에 따라 다양한 방식으로 시작할 수 있습니다.
우리는 Hugging Face Spaces, Colab, 그리고 로컬에서 UI를 제공합니다!

### Hugging Face Spaces

Hugging Face Spaces에서 UI를 시작하려면, 다음 링크를 클릭하면 됩니다:

[![Deploy on Spaces](https://huggingface.co/datasets/huggingface/badges/resolve/main/deploy-on-spaces-md.svg)](https://huggingface.co/login?next=/spaces/autotrain-projects/autotrain-advanced?duplicate=true)

space를 비공개로 유지하고 적절한 하드웨어를 space에 연결해야 합니다.
또한 홈페이지에서 AutoTrain에 대해 더 많이 읽고 링크를 따라가 Hugging Face Spaces에서 자신만의 훈련 인스턴스를 시작할 수 있습니다. 홈페이지를 방문하려면 [여기를 클릭하세요](https://huggingface.co/autotrain).

### Colab

Colab에서 UI를 시작하려면, 다음 링크를 클릭하면 됩니다:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/huggingface/autotrain-advanced/blob/main/colabs/AutoTrain.ipynb)

Colab에서 앱을 실행하려면 ngrok 토큰이 필요합니다. [ngrok](https://ngrok.com/)에서 무료로 가입하면 토큰을 얻을 수 있습니다.
Colab은 포트를 인터넷에 직접 노출할 수 없기 때문입니다.

### 로컬

autotrain 앱을 로컬에서 실행하려면, autotrain-advanced 파이썬 패키지를 설치하세요:

```bash
pip install autotrain-advanced
```

그리고 다음 명령을 실행하세요:

```bash
export HF_TOKEN=your_hugging_face_write_token
autotrain app --host 127.0.0.1 --port 8000
```

이렇게 하면 앱이 `http://127.0.0.1:8000`에서 시작됩니다.

AutoTrain은 pytorch, torchaudio, torchvision 또는 다른 종속성을 자동으로 설치하지 않습니다. 따라서 별도로 설치해야 합니다.
그래서 conda 환경을 사용하는 것이 좋습니다:

```bash
conda create -n autotrain python=3.10
conda activate autotrain

pip install autotrain-advanced

conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
conda install -c "nvidia/label/cuda-12.1.0" cuda-nvcc
conda install xformers -c xformers

python -m nltk.downloader punkt
pip install flash-attn --no-build-isolation
pip install deepspeed

export HF_TOKEN=your_hugging_face_write_token
autotrain app --host 127.0.0.1 --port 8000
```

AutoTrain은 최신 머신러닝 기술을 활용하여 다양한 AI 모델을 쉽게 훈련할 수 있게 해주는 강력한 도구입니다. CLI나 UI를 통해 언어 모델, 이미지 분류 모델, 텍스트 분류 모델 등 다양한 모델을 손쉽게 훈련할 수 있습니다.

AutoTrain을 사용하기 위해서는 먼저 autotrain-advanced 파이썬 패키지를 설치해야 합니다. 그리고 pytorch, torchaudio, torchvision 등의 의존성 패키지도 별도로 설치해야 합니다. conda 환경을 사용하여 이를 관리하는 것이 좋습니다.

AutoTrain은 Hugging Face Spaces, Colab, 그리고 로컬에서 실행할 수 있습니다. Hugging Face Spaces에서는 간단히 링크를 클릭하여 AutoTrain을 시작할 수 있습니다. Colab에서는 ngrok 토큰이 필요합니다. 로컬에서는 CLI 명령어를 사용하여 AutoTrain을 실행할 수 있습니다.

AutoTrain은 최신 AI 기술을 누구나 손쉽게 활용할 수 있도록 도와줍니다. 코딩 지식이 없어도 다양한 AI 모델을 훈련하고 배포할 수 있어, AI의 대중화에 큰 기여를 할 것으로 기대됩니다.

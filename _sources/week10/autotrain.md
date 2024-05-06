# AutoTrain Advanced

🤗 AutoTrain Advanced (혹은 간단히 AutoTrain)은 자연어 처리(NLP), 컴퓨터 비전(CV), 음성, 그리고 테이블 데이터를 위한 최신 모델을 코드 없이 훈련시킬 수 있는 도구입니다. 이는 Hugging Face 팀에서 개발한 훌륭한 도구들을 기반으로 만들어졌으며, 사용하기 쉽도록 설계되었습니다.

## AutoTrain은 누가 사용해야 할까요?

AutoTrain은 NLP, CV, 음성, 또는 테이블 데이터 작업을 위해 최신 모델을 훈련시키고 싶지만, 모델 훈련의 기술적인 세부사항에 시간을 쓰고 싶지 않은 사람들을 위한 것입니다.

AutoTrain은 또한 사용자 정의 데이터셋에 대해 모델을 훈련시키고 싶지만, 역시 모델 훈련의 기술적인 세부사항에 시간을 쓰고 싶지 않은 사람들을 위한 것입니다.

우리의 목표는 누구나 쉽게 어떤 작업에 대해서도 최신 모델을 훈련시킬 수 있도록 만드는 것이며, 우리의 초점은 데이터 과학자나 머신러닝 엔지니어뿐만 아니라 비기술 사용자에게도 맞춰져 있습니다.

## AutoTrain은 어떻게 사용할까요?

AutoTrain을 사용할 수 있는 여러 가지 방법이 있습니다:

- 코드를 사용하지 않는 사용자는 AutoTrain Docker 이미지로 새 공간을 만들어 `AutoTrain Advanced`를 사용할 수 있습니다:

AutoTrain 공간을 만들려면 [여기를 클릭하세요](https://huggingface.co/login?next=/spaces/autotrain-projects/autotrain-advanced?duplicate=true).

공간을 비공개로 유지하고 적절한 하드웨어를 공간에 연결하는 것을 잊지 마세요.

- 개발자는 파이썬 API를 사용하거나 AutoTrain Advanced UI를 로컬에서 실행하여 AutoTrain에 액세스하고 이를 기반으로 구축할 수 있습니다.

파이썬 API는 `autotrain-advanced` 패키지에서 사용할 수 있습니다.

pip를 사용하여 설치할 수 있습니다:

```bash
pip install autotrain-advanced
```

## AutoTrain을 로컬에서 실행하기

autotrain 앱을 로컬에서 실행하려면 다음 명령을 사용할 수 있습니다:

```bash
export HF_TOKEN=your_hugging_face_write_token
autotrain app --host 127.0.0.1 --port 8000
```

이렇게 하면 `http://127.0.0.1:8000`에서 앱이 시작됩니다.

다른 패키지와의 충돌을 피하기 위해 가상 환경에 autotrain-advanced를 설치하는 것이 좋습니다.

```bash
conda create -n autotrain python=3.10
conda activate autotrain
pip install autotrain-advanced
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
conda install -c "nvidia/label/cuda-12.1.0" cuda-nvcc
export HF_TOKEN=your_hugging_face_write_token
autotrain app --host 127.0.0.1 --port 8000
```

AutoTrain은 최신 머신러닝 모델을 활용하여 다양한 데이터 유형(텍스트, 이미지, 음성, 테이블 등)에 대해 강력한 AI 모델을 쉽게 만들 수 있게 해줍니다. 코딩 지식이 없어도 사용할 수 있으며, 데이터 과학자나 엔지니어가 아닌 일반 사용자도 고품질의 AI 모델을 손쉽게 만들 수 있습니다.

AutoTrain을 사용하면 복잡한 기술적 세부사항을 몰라도 최신 머신러닝 기술의 장점을 활용할 수 있습니다. 또한 사용자 정의 데이터에 맞춘 맞춤형 AI 모델을 손쉽게 개발할 수 있어, 다양한 비즈니스와 연구 분야에서 AI를 활용하는데 큰 도움이 될 것입니다.

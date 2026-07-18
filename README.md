# cosmos_setup


# 手順１:コンテナの起動と動画生成デモ
cosmos3のgithub,quickstartから:Generator with vLLM-Omni

## コンテナとvllmサーバーの起動
```
# vllmサーバーの起動
docker run --runtime nvidia --gpus all \
  -e HF_TOKEN=<実際のトークンで置き換える> \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  -v "$(pwd):/workspace" \
  -p 8000:8000 \
  --ipc=host \
  vllm/vllm-omni:cosmos3 \
  vllm serve nvidia/Cosmos3-Nano \
  --omni \
  --model-class-name Cosmos3OmniDiffusersPipeline \
  --allowed-local-media-path / \
  --port 8000
```

## text2videoのデモ
```
# デモの実行
time curl -sS -X POST http://localhost:8000/v1/videos/sync \
  --form-string "prompt=A robotic arm on an industrial assembly line picks up a metal part and places it on a conveyor belt." \
  --form-string "negative_prompt=blurry, distorted, low quality" \
  --form-string "size=832x480" \
  --form-string "num_frames=300" \
  --form-string "fps=24" \
  --form-string "num_inference_steps=35" \
  --form-string "guidance_scale=4.0" \
  --form-string "seed=42" \
  -o smoke_t2v.mp4
```

## image2videoのデモ
```
time curl -sS -X POST http://localhost:8000/v1/videos/sync \
  --form-string "prompt=The machine in the image continues its normal periodic operation. Static camera, consistent lighting." \
  --form-string "negative_prompt=blurry, distorted, low quality" \
  --form-string "size=832x480" \
  --form-string "num_frames=81" \
  --form-string "fps=24" \
  --form-string "num_inference_steps=35" \
  --form-string "guidance_scale=4.0" \
  --form-string "seed=42" \
  -w "\nHTTP %{http_code}\n" \
  -F "input_reference=@/home/nvidia/NVIDIA_SAMP/cosmos3/workspace/IPAD_dataset/R02/training/frames/02/025.jpg" \
  -o outputs/i2v_R02_train_s42.mp4
```

##

```
docker exec -it vad-work bash
```


# VideoMAE関連の用意
## NGCへログイン
```
docker login nvcr.io
```
```
Username: $oauthtoken
Password: <NGC Personal API Key>
```

## imageの取得とコンテナの作成
```
docker pull nvcr.io/nvidia/pytorch:26.06-py3
```
```
mkdir -p ./cosmos3-videomae/{src,data,outputs,cache}
```
```
docker run -it \
  --name videomae \
  --gpus all \
  --ipc=host \
  -v "$HOME/research/nvidia_2026summer:/workspace/videomae" \
  -w /workspace/videomae \
  nvcr.io/nvidia/pytorch:25.10-py3
```


## 最低限のパッケージを導入
```
python -m pip install \
  "transformers==4.56.2" \
  accelerate \
  av
```

# VideoMAEの用意2
## ディレクトリ構造
```
nvidia_2026summer/
├── configs/       実験設定
├── data/
│   ├── raw/       展開前・未加工dataset
│   ├── ucf101/    UCF101本体
│   ├── splits/    train/validation/testの一覧
│   └── metadata/  動画情報・集計結果
├── environment/   環境情報
├── logs/          学習log
├── outputs/
│   ├── checkpoints/
│   ├── metrics/
│   ├── predictions/
│   └── figures/
├── scripts/       実行script
├── src/
│   ├── datasets/
│   ├── models/
│   ├── training/
│   └── utils/
└── tests/         小規模動作確認
```
```
cd ~/research/nvidia_2026summer

mkdir -p \
  configs \
  data/raw \
  data/ucf101 \
  data/splits \
  data/metadata \
  logs \
  outputs/checkpoints \
  outputs/metrics \
  outputs/predictions \
  outputs/figures \
  scripts \
  src/datasets \
  src/models \
  src/training \
  src/utils \
  tests
```

## データセット（UCF101）
```
curl -fL \
  --retry 5 \
  --retry-all-errors \
  --retry-delay 5 \
  -C - \
  -o UCF-101.zip \
  "https://huggingface.co/datasets/quchenyuan/UCF101-ZIP/resolve/main/UCF-101.zip?download=true"
```
```
curl -fL \
  --retry 5 \
  --retry-all-errors \
  --retry-delay 5 \
  -C - \
  -o UCF101TrainTestSplits-RecognitionTask.zip \
  "https://huggingface.co/datasets/quchenyuan/UCF101-ZIP/resolve/main/UCF101TrainTestSplits-RecognitionTask.zip?download=true"
```
```
cd ~/research/nvidia_2026summer

unzip data/raw/UCF-101.zip -d data/ucf101/

unzip \
  data/raw/UCF101TrainTestSplits-RecognitionTask.zip \
  -d data/splits/
```


## コンテナ入る
```
docker exec -it \
  --user "$(id -u):$(id -g)" \
  --env HOME=/workspace/videomae/.home \
  --env HF_HOME=/workspace/videomae/.cache/huggingface \
  --env PYTHONPATH=/workspace/videomae \
  -w /workspace/videomae \
  videomae \
  bash
```

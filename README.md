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


# VideoMAEの用意
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
  -v "./cosmos3-videomae/:/workspace/videomae" \
  -w /workspace/videomae \
  nvcr.io/nvidia/pytorch:26.06-py3
```

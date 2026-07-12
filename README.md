# cosmos_setup


# 手順１:コンテナの起動と動画生成デモ
cosmos3のgithub,quickstartから:Generator with vLLM-Omni

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

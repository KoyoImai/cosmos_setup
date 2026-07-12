# cosmos_setup

# 手順０：huggingfaceでアクセストークンを作成し、認証する
コンテナ起動後に認証でも大丈夫らしい

# 手順１:こんてな起動
cosmos3のgithub,quickstartから:Generator with vLLM-Omni

```
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

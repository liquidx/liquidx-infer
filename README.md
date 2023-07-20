```
python3 \
 -m llama_cpp.server \
 --model $MODEL \
 --n_gpu_layers 1 \
 --port 19000
```

# Example Prompts:

```
context: Alastair lives in Tokyo
meta:
- country: japan
###
context: Patricia loves reading
meta:
- hobby: books
###
context: Simon lives in San Francisco
meta:
```

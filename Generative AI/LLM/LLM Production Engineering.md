```
User
 ↓
API Gateway
 ↓
LLM Service
 ↓
Cache
 ↓
Model Server
 ↓
GPU
 ↓
Response
```

1. LLM Serving:
2. vLLM (Most Important):
   Designed for:
	1. Fast inference
	2. High throughput
	3. Low latency
	4. Better GPU utilization
   >Paged Attention is a memory management technique used by vLLM that stores KV cache in non-contiguous memory pages, reducing fragmentation and improving GPU utilization.
3. KV Cache
   ```
   For question like weather.
   Transformer stores K,V Pairs.
   without it has to recompute everything.
   With KV
   Store previous K,V
   Reuse them
   ```
4. Continuous Batching
   Benefits:
	- Higher throughput
	- Lower latency
	- Better utilization
   Traditional batching:
   ```
   Batch formed
		↓
		Run
		↓
		Wait
		↓
		Next batch
   ```
   Continuous Batching:
   ```
   Request arrives
	↓
	Immediately joins active batch
   ```
5. Nvidia has TensorRT-LLM for maximum inference on Nvidia's GPUs
6. TGI:- Production model serving
   Features:
	- Token streaming
	- Batching
	- Metrics
	- OpenAI-compatible APIs
7. Quantisation: Reduce numerical precision.
8. FP32:
	Most accurate.
	Most expensive.
9. FP16:
	Half memory.
    Widely used.
    GPTQ: Generative Pretrained Transformer Quantization.
    AWQ
    QLoRA
    Fine-Tuning: Adapt model behaviour
10. Cache: 
	1. Prompt Cache
	2. Semantic Cache
11. We monitor:
	1. Latency
	2. Token usage
	3. Cost
	4. Hallucination
	5. User Feedback

```
User
 ↓
API Gateway
 ↓
Semantic Cache
 ↓
Agent
 ↓
RAG
 ↓
vLLM / TGI / TensorRT-LLM
 ↓
LLM
 ↓
Response
 ↓
Langfuse
Helicone
Phoenix
```



>vLLM
	→ High-performance LLM serving`
Paged Attention
	→ Efficient KV cache memory management
Continuous Batching
→ Dynamic batching for better GPU utilization
TensorRT-LLM
→ NVIDIA optimized inference
TGI
→ Hugging Face serving framework
Quantization
→ Reduce precision for lower memory and faster inference
FP16
→ 16-bit
INT8
→ 8-bit
INT4
→ 4-bit
GPTQ
→ Post-training quantization
AWQ
→ Activation-aware quantization
LoRA
→ Train small adapters
QLoRA
→ Quantized LoRA
Prompt Cache
→ Exact prompt reuse
Semantic Cache
→ Similar query reuse
Observability
→ Monitor latency, cost, token usage, hallucinations
Langfuse / Helicone / Phoenix
→ Production monitoring and evaluation


For handling production scale traffic:-

- More servers/pods
- Load balancing
- Caching
- Batching
- Queues
- Database optimization


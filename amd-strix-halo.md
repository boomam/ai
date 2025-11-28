# System Specs
* Framework Desktop Motherboard - AMD Ryzen AI Max 395+ 128Gb
* Samsung 990 Pro 1Tb (Boot/Root) - ext4
* Western Digital SN5000 4TB (Data) - zfs (8Gb ARC)
* Noctua NF-A12-25 G2 on CPU (push)
* Noctua NF-A12-25 G2 Case Fan (Intake)
* Jonbo C6-ITX Case
* Corsair SF600 PSU

# Software Setup
* Ubuntu 25.10
* ROCM 7.1.0
* Docker 29.0.4
* Additional Apps -
  * Patchmon Agent
  * Beszel Agent
  * Promtail

# Ollama Deployment
Ollama docker compose -  

```yaml
version: '3.8'
name: ollama
services:
  ollama:
#    image: ollama/ollama:0.13.0-rocm
    image: ollama/ollama:0.13.0
    container_name: ollama
    restart: always 
    ports:
      - "11434:11434"
    volumes:
      - /mnt/llm/ollama-general:/root/.ollama
    environment:
      - OLLAMA_HOST:0.0.0.0:11434      
      - OLLAMA_CONTEXT_LENGTH=8192
      - OLLAMA_KEEP_ALIVE=24h
      - OLLAMA_VULKAN=1 #required for vulkan detection on standard image
      - GGML_VK_VISIBLE_DEVICES=0 #required for vulkan detection on standard image
    devices:
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    networks:
      - proxynet

networks:
  proxynet:
    name: proxynet
    external: true      
```
&nbsp;  

# Test Proceadure
`docker exec -it ollama sh`  

* `ollama run llama3.1:8b --verbose`
* `ollama run llama3.1:70b --verbose`
* `ollama run gpt-oss:120b --verbose`

First prompt: `hello` - not measured.  
Second prompt: `write a 1000 story about cats who are learning kubernetes`- measured  

# Results

| Model | Num Parameters | ROCM Prompt Eval Rate | ROCM Eval Rate | Vulkan Prompt Eval Rate | Vulkan Eval Rate |
| ----- | ----- | ----- | ----- | ----- | ----- |
| llama3.1 | 8B | 1165.34 | 35.75	|	780.07 | 41.87 |
| llama3.1 | 70B | 154.05 | 4.32 | 88.94 | 5.05 |
| gpt-oss | 120B | 246.79 | 33.88	| 331.36 | 32.73 |

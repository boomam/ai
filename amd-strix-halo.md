# System Specs  
## Hardware  
* Framework Desktop Motherboard - AMD Ryzen AI Max 395+ 128Gb
  * 96GB set in BIOS for APU/GPU 
* Samsung 990 Pro 1Tb (Boot/Root)
  * ext4
* Western Digital SN5000 4TB (Data)
  * zfs (8Gb ARC)
* Noctua NF-A12-25 G2 on CPU (push)
  * 40% minimum until 60*c
* Noctua NF-A12-25 G2 Case Fan (Intake, bottom)
  * 40% minimum until 60*c
* Jonbo C6-ITX Case
* Corsair SF600 PSU

## Software Setup  
_Always assumed to be latest releases at time of tests_  
* Ubuntu 25.10
* ROCM x.x.x - versions per test run noted below
* Docker x.x.x - versions per test run noted below
* Additional Apps -
  * Patchmon Agent
  * Beszel Agent
  * Promtail

# Stack Deployment  
## Ollama Deployment  
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
      - /mnt/llm/ollama:/root/.ollama
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
## ROCM/AMD GPU Driver Installation.
1. Full update Ubuntu.
2. Check latest wget paths here: https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/quick-start.html
3. `wget https://repo.radeon.com/amdgpu-install/7.1/ubuntu/noble/amdgpu-install_7.1.70100-1_all.deb`
4. `apt install ./amdgpu-install_7.1.70100-1_all.deb -y`
5. `amdgpu-install`
6. `reboot now`
   
# Test Proceadure
`docker exec -it ollama sh`  

* `ollama run llama3.1:8b --verbose`
* `ollama run llama3.1:70b --verbose`
* `ollama run gpt-oss:120b --verbose`

First prompt: `hello` - not measured.  
Second prompt: `write a 1000 story about cats who are learning kubernetes`- measured  

# Results
## ROCM 7.1.0
ROCM 7.1.0, Docker 29.0.4, Ollama 0.13.0

| Model | Num Parameters | ROCM Prompt Eval Rate | ROCM Eval Rate | Vulkan Prompt Eval Rate | Vulkan Eval Rate |
| ----- | ----- | ----- | ----- | ----- | ----- |
| llama3.1 | 8B | 1165.34 | 35.75	|	780.07 | 41.87 |
| llama3.1 | 70B | 154.05 | 4.32 | 88.94 | 5.05 |
| gpt-oss | 120B | 246.79 | 33.88	| 331.36 | 32.73 |

## ROCM 7.1.1
ROCM 7.1.1, Docker 29.1.1, Ollama 0.13.0

| Model | Num Parameters | ROCM Prompt Eval Rate | ROCM Eval Rate | Vulkan Prompt Eval Rate | Vulkan Eval Rate |
| ----- | ----- | ----- | ----- | ----- | ----- |
| llama3.1 | 8B |  | 	|	581.91 | 41.65 |
| llama3.1 | 70B |  |  | 87.98 | 5.04 |
| gpt-oss | 120B |  | 	| 361.93 | 32.91 |

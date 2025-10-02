# shadeform-ai
Repository to capture commands and scripts to manage GPU instance and volumes with ShadeForm

### API Key

Store the Shadeform AI API key in an environment variable `SHADEFORM_AI_API_KEY`

    export SHADEFORM_AI_API_KEY='<<KEY>>'

### SSH Private Key

Store the Shadeform AI SSH Private Key in file `scratch/shadeform_ai_private_key.pem`

Change the permission of the private key as `chmod 600 scratch/shadeform_ai_private_key.pem`


### Find cheapest instance

    curl -s --request GET \
        --url 'https://api.shadeform.ai/v1/instances/types?available=true&sort=price' \
        --header "X-API-KEY: $SHADEFORM_AI_API_KEY" | jq -r '.instance_types[0]'

### Find GPU instances by type, sort by price and find cheapest instance

    curl -s --request GET \
        --url 'https://api.shadeform.ai/v1/instances/types?gpu_type=A6000&available=true&sort=price' \
        --header "X-API-KEY: $SHADEFORM_AI_API_KEY" | jq -r '.instance_types[0]'

### Create Instance without running any process

    curl --request POST \
        --url 'https://api.shadeform.ai/v1/instances/create' \
        --header "X-API-KEY: $SHADEFORM_AI_API_KEY" \
        --header 'Content-Type: application/json' \
        --data '{
                    "cloud": "imwt",
                    "region": "desmoines-usa-2",    
                    "shade_instance_type": "A6000",
                    "shade_cloud": true,
                    "name": "quickstart"
                }'

##### List instances

    curl --request GET --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances' | jq .

##### Wait for instance to become active

    curl --request GET --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances/{instance_id}/info' | jq .

##### SSH into the instance

    ssh -o "StrictHostKeyChecking no" -i scratch/shadeform_ai_private_key.pem shadeform@{instance_ip}

##### UFW firewall rules

    sudo ufw status

    sudo ufw default deny incoming

    sudo ufw allow 22

    sudo ufw allow OpenSSH

    sudo ufw deny 80/tcp

    sudo ufw --force enable

    sudo ufw status

##### Check os, cpu, memory and gpu info

    uname -a

    cat /etc/os-release
    
    cat /proc/cpuinfo

    grep MemTotal /proc/meminfo

    sudo apt-get install -y gpustat

    gpustat

##### Monitoring cpu and gpu stats

    top

    while sleep 1; do gpustat; done

### Run Jupyter Notebook

Create SSH Tunnel with port forwarding

    ssh -o "StrictHostKeyChecking no" -i scratch/shadeform_ai_private_key.pem -L 8888:localhost:8888 shadeform@{instance_ip}

Create a directory for jupyter

    mkdir -p /home/shadeform/jupyter

    mkdir -p /home/shadeform/jupyter/start-notebook.d

    echo '#!/bin/sh' >> /home/shadeform/jupyter/start-notebook.d/jupytext-install.sh
    
    echo 'pip install jupytext' >> /home/shadeform/jupyter/start-notebook.d/jupytext-install.sh
    
    chmod -R a+rwx /home/shadeform/jupyter


Start minimal jupyter notebook using docker

    docker run -d --rm --name pytorch-notebook --network=host --ipc=host --runtime nvidia --gpus all -v "/home/shadeform/jupyter/start-notebook.d:/usr/local/bin/start-notebook.d" -v "/home/shadeform/jupyter:/home/work" -w "/home/work" quay.io/jupyter/minimal-notebook:python-3.12

Start pytorch cuda conda jupyter notebook using docker

    docker run -d --rm --name pytorch-notebook --network=host --ipc=host --runtime nvidia --gpus all -v "/home/shadeform/jupyter/start-notebook.d:/usr/local/bin/start-notebook.d" -v "/home/shadeform/jupyter:/home/work" -w "/home/work" quay.io/jupyter/pytorch-notebook:cuda12-conda-25.5.1

Get the notebook token from docker logs

    docker logs --follow pytorch-notebook | grep 'http://localhost:8888/lab?token'

Access the jupyter notebook in browser via localhost through the ssh tunnel. Use the `token` to login

    http://localhost:8888

Sample python script to test the notebook:

```python
import torch

# Check if CUDA is available
if torch.cuda.is_available():
  print("CUDA is available. GPU Details:")
  # Number of GPUs available
  print(f"Number of GPUs: {torch.cuda.device_count()}")
  for i in range(torch.cuda.device_count()):
    # Get the name of the current GPU
    print(f"GPU {i}: {torch.cuda.get_device_name(i)}")
else:
  print("CUDA is not available. Running on CPU.")
```

Stop the docker container running jupyter notebook

    docker stop pytorch-notebook

Upload file from local to shadeform

    scp -r -i scratch/shadeform_ai_private_key.pem local_sample_python.py shadeform@{instance_ip}:/home/shadeform/jupyter
    
Download file from shadeform to local

    scp -r -i scratch/shadeform_ai_private_key.pem shadeform@{instance_ip}:/home/shadeform/jupyter/lora_model/ /Users/harishkannarao/Downloads/shadeform_download

    scp -r -i scratch/shadeform_ai_private_key.pem shadeform@{instance_ip}:/home/shadeform/jupyter/model/ /Users/harishkannarao/Downloads/shadeform_download

    scp -r -i scratch/shadeform_ai_private_key.pem shadeform@{instance_ip}:/home/shadeform/jupyter/gguf_model/ /Users/harishkannarao/Downloads/shadeform_download

### Run model(s) using vllm

Create SSH Tunnel with port forwarding

    ssh -i scratch/shadeform_ai_private_key.pem -L 8000:localhost:8000 shadeform@{instance_ip}

Start AI LLM Model using vllm through docker

    docker run -d --rm --name vllm-openai -p 8000:8000 --runtime nvidia --gpus all --env VLLM_API_KEY=SAMPLE_SECRET_FROM_VAULT vllm/vllm-openai:latest --gpu-memory-utilization 0.7 --model openai/gpt-oss-20b

Follow the vllm logs in docker container

    docker logs --follow vllm-openai

Check the response from vllm

```bash

curl --request POST \
--url 'http://localhost:8000/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header "Authorization: Bearer SAMPLE_SECRET_FROM_VAULT" \
--data '{
  "model": "openai/gpt-oss-20b",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Who won the world series in 2020?"}
  ]
}'

```

Stop the vllm docker container

    docker stop vllm-openai

Create SSH Tunnel with port forwarding for embedding model

    ssh -i scratch/shadeform_ai_private_key.pem -L 8001:localhost:8001 shadeform@{instance_ip}

Start AI LLM Model using vllm through docker

    docker run -d --rm --name vllm-openai-embedding -p 8001:8000 --runtime nvidia --gpus all --env VLLM_API_KEY=SAMPLE_SECRET_FROM_VAULT vllm/vllm-openai:latest --gpu-memory-utilization 0.3 --model mixedbread-ai/mxbai-embed-large-v1

Follow the vllm logs in docker container

    docker logs --follow vllm-openai-embedding

Stop the vllm docker container

    docker stop vllm-openai-embedding

### Run model using ollama

Create SSH Tunnel with port forwarding

    ssh -i scratch/shadeform_ai_private_key.pem -L 11434:localhost:11434 shadeform@{instance_ip}

Start Ollama through docker

    docker run -d --rm --name ollama -v ollama:/root/.ollama --network=host --ipc=host --runtime nvidia --gpus all ollama/ollama

Follow the vllm logs in docker container

    docker logs --follow ollama

Go into the ollama container

    docker exec -it ollama /bin/bash

    ollama --version

Stop the vllm docket container

    docker stop ollama

### Restart the instance

    curl -v --request POST --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances/{instance_id}/restart' 

### Delete the instance

    curl -v --request POST --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances/{instance_id}/delete'


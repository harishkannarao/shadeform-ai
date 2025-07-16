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
                    "cloud": "hyperstack",
                    "region": "oslo-norway-1",    
                    "shade_instance_type": "A4000",
                    "shade_cloud": true,
                    "name": "quickstart"
                }'

##### List instances

    curl --request GET --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances' | jq .

##### Wait for instance to become active

    curl --request GET --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances/{instance_id}/info' | jq .

##### SSH into the instance

    ssh -i scratch/shadeform_ai_private_key.pem shadeform@{instance_ip}

##### Check os, cpu, memory and gpu info

    uname -a

    cat /etc/os-release
    
    cat /proc/cpuinfo

    grep MemTotal /proc/meminfo

    sudo apt-get install gpustat

    gpustat

##### Monitoring cpu and gpu stats

    top

    while sleep 1; do gpustat; done

### Run Jupyter Notebook

Create SSH Tunnel with port forwarding

    ssh -i scratch/shadeform_ai_private_key.pem -L 8888:localhost:8888 shadeform@{instance_ip}

Start jupyter notebook using docker

    docker run -d --rm --name pytorch-notebook --network=host --ipc=host --runtime nvidia --gpus all quay.io/jupyter/pytorch-notebook:cuda12-python-3.11.8

Get the notebook token from docker logs

    docker logs --follow pytorch-notebook | grep 'token'

Access the jupyter notebook in browser via localhost through the ssh tunnel. Use the `token` to login

    http://127.0.0.1:8888

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

Stop the docket container running jupyter notebook

    docker stop pytorch-notebook


### Run model using vllm

### Restart the instance

    curl -v --request POST --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances/{instance_id}/restart' 

### Delete the instance

    curl -v --request POST --header "X-API-KEY: $SHADEFORM_AI_API_KEY" --url 'https://api.shadeform.ai/v1/instances/{instance_id}/delete'

### UFW firewall rules

    sudo ufw status

    sudo ufw default deny incoming

    sudo ufw allow 22

    sudo ufw allow OpenSSH

    sudo ufw deny 80/tcp

    sudo ufw enable

    sudo ufw status
    
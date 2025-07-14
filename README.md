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

### Create Instance to run Jupyter Notebook

### Create Instance to run model using vllm

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
    
# shadeform-ai
Repository to capture commands and scripts to manage GPU instance and volumes with ShadeForm

### API Key

Store the Shadeform AI API key in an environment variable `SHADEFORM_AI_API_KEY`

    export SHADEFORM_AI_API_KEY='<<KEY>>'

### SSH Private Key

Store the Shadeform AI SSH Private Key in file `scratch/shadeform_ai_private_key.pem`


### Find GPU instances by type and sort by price

    curl -s --request GET \
        --url 'https://api.shadeform.ai/v1/instances/types?gpu_type=A6000&num_gpus=1&available=true&sort=price' \
        --header "X-API-KEY: $SHADEFORM_AI_API_KEY" | jq .

### UFW firewall rules

    sudo ufw status

    sudo ufw default deny incoming

    sudo ufw allow 22

    sudo ufw allow OpenSSH

    sudo ufw allow from your_ip_address to any port 22

    sudo ufw enable

    sudo ufw status
    
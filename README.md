# shadeform-ai
Repository to capture commands and scripts to manage GPU instance and volumes with ShadeForm

### API Key

Store the shadeform ai API key in an environment variable `SHADEFORM_AI_API_KEY`

    export SHADEFORM_AI_API_KEY='<<KEY>>'

### Find GPU instances by type and sort by price

    curl -s --request GET \
        --url 'https://api.shadeform.ai/v1/instances/types?gpu_type=A6000&num_gpus=1&available=true&sort=price' \
        --header "X-API-KEY: $SHADEFORM_AI_API_KEY" | jq .
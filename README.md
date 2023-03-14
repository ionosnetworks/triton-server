# triton-server
Instructions on running triton inference server

## Starting the server

docker run --gpus all --rm --ipc=host --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -p8000:8000 -p8001:8001 -p8002:8002 -v ~/triton/models:/models nvcr.io/nvidia/tritonserver:22.06-py3 tritonserver --model-repository=/models --strict-model-config=false

## YOLOV7
put below in yolov7/config.pbtxt

name: "yolov7"
platform: "tensorrt_plan"
max_batch_size: 8
dynamic_batching { }

### Folder structure
 
triton/
└── models
    └── yolov7
        ├── 1
        │   └── model.plan
        └── config.pbtxt
 
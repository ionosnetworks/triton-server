# YOLOv7 on Triton Inference Server Instructions
Instructions to deploy YOLOv7 as TensorRT engine to [Triton Inference Server](https://github.com/NVIDIA/triton-inference-server).


## Export TensorRT

```bash
#install onnx-simplifier not listed in general yolov7 requirements.txt
pip3 install onnx-simplifier 

# Pytorch Yolov7 -> ONNX with grid, EfficientNMS plugin and dynamic batch size
python export.py --weights ./yolov7.pt --grid --end2end --dynamic-batch --simplify --topk-all 100 --iou-thres 0.65 --conf-thres 0.35 --img-size 640 640
# ONNX -> TensorRT with trtexec and docker
docker run -it --rm --gpus=all nvcr.io/nvidia/tensorrt:22.06-py3
# Copy onnx -> container: 
docker cp yolov7.onnx <container-id>:/workspace/
# Export with FP16 precision, min batch 1, opt batch 8 and max batch 8
./tensorrt/bin/trtexec --onnx=yolov7.onnx --minShapes=images:1x3x640x640 --optShapes=images:8x3x640x640 --maxShapes=images:8x3x640x640 --fp16 --workspace=4096 --saveEngine=yolov7-fp16-1x8x8.engine --timingCacheFile=timing.cache
# Test engine
./tensorrt/bin/trtexec --loadEngine=yolov7-fp16-1x8x8.engine
# Copy engine -> host: 
docker cp <container-id>:/workspace/yolov7-fp16-1x8x8.engine .
```

## Model Repository

See [Triton Model Repository Documentation](https://github.com/triton-inference-server/server/blob/main/docs/model_repository.md#model-repository) for more info.

```bash
# Create folder structure
mkdir -p triton/models/yolov7/1/
touch triton/models/yolov7/config.pbtxt
# Place model
mv yolov7-fp16-1x8x8.engine triton/models/yolov7/1/model.plan
```

## Model Configuration
 
Minimal configuration for `triton/models/yolov7/config.pbtxt`:

```
name: "yolov7"
platform: "tensorrt_plan"
max_batch_size: 8
dynamic_batching { }
```

Example repository:

```bash
$ tree triton/
triton/
└── models
    └── yolov7
        ├── 1
        │   └── model.plan
        └── config.pbtxt

3 directories, 2 files
```

## Starting the server

```
docker run --gpus all --rm --ipc=host --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -p8000:8000 -p8001:8001 -p8002:8002 -v ~/triton/models:/models nvcr.io/nvidia/tritonserver:22.06-py3 tritonserver --model-repository=/models --strict-model-config=false
```
 

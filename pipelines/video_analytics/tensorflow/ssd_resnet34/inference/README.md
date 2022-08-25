# **Tensorflow SSD-ResNet34 INFERENCE - VDMS-Streamer**

## **Description**

This Pipeline provides the containerized implementation of the [VDMS-Streamer](https://github.com/intel-innersource/frameworks.ai.end2end-ai-pipelines.video-streamer/tree/8296ae20bff344bfb5c8c55ee484d9d385eceed2) project.

The workload aims to implement end-to-end video streamer pipeline involving media and analytics segments using GStreamer and TensorFlow. The pipeline handles video decode and processing followed by  object detection and classification using the ssd-mobilenet-resnet-34 model using TensorFlow for single and multiple instances in FP32, BF16 and INT8 precisions. The metadata generated is uploaded to VDMS.

## **Project Structure**
```
├── vdms-streamer @ v0.1.0
├── Dockerfile.video-streamer
├── Makefile
├── README.md
└── docker-compose.yml
```
[*Makefile*](https://github.com/intel-innersource/frameworks.ai.infrastructure.machine-learning-operations/blob/sramakr1/vdms-streamer/pipelines/video_analytics/tensorflow/ssd_resnet34/inference/Makefile)

```
FINAL_IMAGE_NAME ?= vdms-video-streamer
KERAS_VERSION ?= 2.10.0.dev2022062207
OUTPUT_DIR ?= /output
TF_ESTIMATOR_VERSION ?= 2.10.0.dev2022062208
TF_VERSION ?= 2.10.0.202226-cp38-cp38
VIDEO_PATH ?= $$(pwd)/mall.avi
VIDEO = $(shell basename ${VIDEO_PATH})
WHL_WW ?= 26
WHL_YR ?= 2022

vdms:
	WHL_WW=${WHL_WW} \
	WHL_YR=${WHL_YR} \
	numactl --physcpubind=51-55 --membind=1 docker compose up -d vdms

video-streamer: vdms
	FINAL_IMAGE_NAME=${FINAL_IMAGE_NAME} \
	KERAS_VERSION=${KERAS_VERSION} \
	OUTPUT_DIR=${OUTPUT_DIR} \
	TF_ESTIMATOR_VERSION=${TF_ESTIMATOR_VERSION} \
	TF_VERSION=${TF_VERSION} \
	VIDEO=${VIDEO} \
	VIDEO_PATH=${VIDEO_PATH} \
	WHL_WW=${WHL_WW} \
	WHL_YR=${WHL_YR} \
	docker compose up video-streamer --build

clean:
	docker compose down 
```

[*docker-compose.yml*](https://github.com/intel-innersource/frameworks.ai.infrastructure.machine-learning-operations/blob/sramakr1/vdms-streamer/pipelines/video_analytics/tensorflow/ssd_resnet34/inference/docker-compose.yml)

```
services:
  vdms:
    image: vuiseng9/intellabs-vdms:demo-191220
    network_mode: "host"
    ports:
      - "55555:55555"
  ## Base
  video-streamer:
    build:
      args:
        KERAS_VERSION: ${KERAS_VERSION}
        TF_ESTIMATOR_VERSION: ${TF_ESTIMATOR_VERSION}
        TF_VERSION: ${TF_VERSION}
        WHL_WW: ${WHL_WW}
        WHL_YR: ${WHL_YR}
        http_proxy: ${http_proxy}
        https_proxy: ${https_proxy}
        no_proxy: ${no_proxy}
      dockerfile: Dockerfile.video-streamer
    command: sh -c "./benchmark.sh  && cp -r ../*.txt ${OUTPUT_DIR}"
    depends_on:
      - vdms
    environment:
      - OUTPUT_DIR=${OUTPUT_DIR}
      - VIDEO_FILE=/workspace/vdms-streamer/${VIDEO}
      - VIDEO_PATH=${VIDEO_PATH}
      - http_proxy=${http_proxy}
      - https_proxy=${https_proxy}
      - no_proxy=${no_proxy}
    healthcheck:
      test: netstat -lnpt | grep 55555 || exit 1
      interval: 10s
      timeout: 5s
      retries: 5
    image: ${FINAL_IMAGE_NAME}:inference-ww${WHL_WW}-${WHL_YR}-centos-8
    network_mode: "host"
    ports:
      - "55555:55555"
    privileged: true
    volumes:
      - ./vdms-streamer:/workspace/vdms-streamer
      - /${OUTPUT_DIR}:${OUTPUT_DIR}
      - /${VIDEO_PATH}:/workspace/vdms-streamer/${VIDEO}
    working_dir: /workspace/vdms-streamer
```

# **VDMS-Streamer** 
End-to-end AI workflow utilizing the VDMS-Streamer. More Information [here](https://github.com/intel-innersource/frameworks.ai.end2end-ai-pipelines.video-streamer/tree/8296ae20bff344bfb5c8c55ee484d9d385eceed2). The pipeline runs the ```benchmark.sh``` script of the [VDMS-Streamer](https://github.com/intel-innersource/frameworks.ai.end2end-ai-pipelines.video-streamer/blob/8296ae20bff344bfb5c8c55ee484d9d385eceed2/benchmark.sh) project.

## **Quick Start**
* Make sure that the enviroment setup pre-requisites are satisfied per the document [here](https://github.com/intel-innersource/frameworks.ai.infrastructure.machine-learning-operations/blob/develop/pipelines/README.md)

* Pull and configure the dependent repo submodule ```git submodule update --init --recursive ```.

* Download [`mall.avi`](http://10.112.229.21/vdms/mall.avi). The default assumes the video is located in the same location as the cloned [VDMS-Streamer Repo](https://github.com/intel-innersource/frameworks.ai.end2end-ai-pipelines.video-streamer/tree/8296ae20bff344bfb5c8c55ee484d9d385eceed2/dataset) folder.

* Other Variables:

Variable Name    | Default             | Notes                                   |
:----------------:|:------------------: | :--------------------------------------:|
FINAL_IMAGE_NAME    | vdms-video-streamer | Final Docker Image Name                 |
KERAS_VERSION       | 2.10.0.dev2022062207| Keras Version                           |
OUTPUT_DIR          | `/output`           | Output Directory                        |
TF_ESTIMATOR_VERSION| 2.10.0.dev2022062208| TF Estimator Version                    |
TF_VERSION          | 2.10.0.202226-cp38-cp38 | TF `.whl` version                      |
VIDEO_PATH          | `$$(pwd)/mall.avi`    | Path of the Video File  |
WHL_WW              |  `26`               |  Work week of TensorFlow `.whl` package |
WHL_YR              | `2022`              | Work year  of Tensorflow `.whl` package |

## **Build and Run**
Build and run with defaults:

  ```$ make video-streamer```

## **Build and Run Example**
```
#1 [internal] load build definition from Dockerfile.video-streamer
#1 transferring dockerfile: 47B done
#1 DONE 0.0s

#2 [internal] load .dockerignore
#2 transferring context: 2B done
#2 DONE 0.0s

#3 [internal] load metadata for docker.io/library/centos:8
#3 DONE 0.4s

#4 [1/9] FROM docker.io/library/centos:8@sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
#4 DONE 0.0s

#5 [internal] load build context
#5 transferring context: 6.25kB done
#5 DONE 0.0s

#6 [2/9] RUN sed -i.bak '/^mirrorlist=/s/mirrorlist=/#mirrorlist=/g' /etc/yum.repos.d/CentOS-Linux-* &&     sed -i.bak 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-* &&     yum distro-sync -y &&     yum --disablerepo '*' --enablerepo=extras swap centos-linux-repos centos-stream-repos -y &&     yum distro-sync -y &&     yum clean all
#6 CACHED

#7 [3/9] RUN yum update -y && yum install -y     python38     python38-pip     which     wget numactl mesa-libGL net-tools &&     yum clean all
#7 CACHED

#8 [4/9] RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh -O miniconda.sh &&     chmod +x miniconda.sh &&     ./miniconda.sh -b -p ~/conda &&     rm ./miniconda.sh &&     ~/conda/bin/conda create -yn vdms-test python=3.8 &&     export PATH=~/conda/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin &&     source activate vdms-test &&     python3 -m pip --no-cache-dir install --upgrade pip     opencv-python==4.5.5.64     protobuf==3.20.1     pyyaml     setuptools     vdms     wheel
#8 CACHED

#9 [5/9] RUN ln -sf $(which python3) /usr/local/bin/python &&     ln -sf $(which python3) /usr/local/bin/python3 &&     ln -sf $(which python3) /usr/bin/python
#9 CACHED

#10 [6/9] COPY /* /tmp/pip3/
#10 DONE 5.8s

#11 [7/9] RUN source activate vdms-test &&     pip install     "tf-estimator-nightly==2.10.0.dev2022042008"     "keras-nightly==2.10.0.dev2022042007"     /tmp/pip3/tf_nightly-2.10.0.202218-cp38-cp38-linux_x86_64.whl &&     conda install -y -c conda-forge gst-libav==1.18.4 gst-plugins-good=1.18.4 gst-plugins-bad=1.18.4 gst-plugins-ugly=1.18.4 gst-python=1.18.4 pygobject=3.40.1 &&     conda clean --all
#11 0.792 Processing /tmp/pip3/tf_nightly-2.10.0.202218-cp38-cp38-linux_x86_64.whl
#11 1.121 Collecting tf-estimator-nightly==2.10.0.dev2022042008
#11 1.161   Downloading tf_estimator_nightly-2.10.0.dev2022042008-py2.py3-none-any.whl (438 kB)
#11 1.176      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 438.9/438.9 kB 36.5 MB/s eta 0:00:00
#11 1.278 Collecting keras-nightly==2.10.0.dev2022042007
#11 1.285   Downloading keras_nightly-2.10.0.dev2022042007-py2.py3-none-any.whl (1.6 MB)
#11 1.318      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.6/1.6 MB 51.7 MB/s eta 0:00:00
#11 1.365 Collecting libclang>=13.0.0
#11 1.370   Downloading libclang-14.0.1-py2.py3-none-manylinux1_x86_64.whl (14.5 MB)
#11 1.537      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 14.5/14.5 MB 78.7 MB/s eta 0:00:00
#11 1.612 Collecting tensorflow-io-gcs-filesystem>=0.23.1
#11 1.626   Downloading tensorflow_io_gcs_filesystem-0.26.0-cp38-cp38-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (2.4 MB)
#11 1.651      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.4/2.4 MB 100.9 MB/s eta 0:00:00
```
...

```
cpu-video-streamer-1  |  
cpu-video-streamer-1  |         _     _              __ _                                      
cpu-video-streamer-1  |  /\   /(_) __| | ___  ___   / _\ |_ _ __ ___  __ _ _ __ ___   ___ _ __ 
cpu-video-streamer-1  |  \ \ / / |/ _` |/ _ \/ _ \  \ \| __| '__/ _ \/ _` | '_ ` _ \ / _ \ '__|
cpu-video-streamer-1  |   \ V /| | (_| |  __/ (_) | _\ \ |_| | |  __/ (_| | | | | | |  __/ |   
cpu-video-streamer-1  |    \_/ |_|\__,_|\___|\___/  \__/\__|_|  \___|\__,_|_| |_| |_|\___|_|  
cpu-video-streamer-1  | 
cpu-video-streamer-1  |  Intel optimized video streaming pipeline based on GSteamer and Tensorflow
cpu-video-streamer-1  | 
cpu-video-streamer-1  |  /root/conda/envs/vdms-test/lib/gstreamer-1.0:/workspace/vdms-streamer/gst-plugin
cpu-video-streamer-1  | 
cpu-video-streamer-1  | (gst-plugin-scanner:109): GStreamer-WARNING **: 17:44:22.414: Failed to load plugin '/root/conda/envs/vdms-test/lib/gstreamer-1.0/libgstmpg123.so': libmpg123.so.0: cannot open shared object file: No such file or directory
cpu-video-streamer-1  | 
cpu-video-streamer-1  | (gst-plugin-scanner:109): GStreamer-WARNING **: 17:44:22.420: Failed to load plugin '/root/conda/envs/vdms-test/lib/gstreamer-1.0/libgstximagesrc.so': libXdamage.so.1: cannot open shared object file: No such file or directory
cpu-video-streamer-1  | 
cpu-video-streamer-1  | (gst-plugin-scanner:109): GStreamer-WARNING **: 17:44:22.436: Failed to load plugin '/root/conda/envs/vdms-test/lib/gstreamer-1.0/libgstmpg123.so': libmpg123.so.0: cannot open shared object file: No such file or directory
cpu-video-streamer-1  | 
cpu-video-streamer-1  | (gst-plugin-scanner:109): GStreamer-WARNING **: 17:44:22.437: Failed to load plugin '/root/conda/envs/vdms-test/lib/gstreamer-1.0/libgstximagesrc.so': libXdamage.so.1: cannot open shared object file: No such file or directory
cpu-video-streamer-1  | + set +x
cpu-video-streamer-1  | + numactl --physcpubind=0-3 --localalloc gst-launch-1.0 filesrc location=dataset/intel_approved.mp4 '!' decodebin '!' videoconvert '!' video/x-raw,format=RGB '!' videoconvert '!' queue '!' gst_detection_tf conf=config/settings.yaml '!' fakesink
cpu-video-streamer-1  | 
cpu-video-streamer-1  | (gst-launch-1.0:174): GStreamer-CRITICAL **: 17:44:24.178: The created element should be floating, this is probably caused by faulty bindings
cpu-video-streamer-1  | INFO:gst_detection_tf:Loading model: models/ssd_resnet34_fp32_1200x1200_pretrained_model.pb
cpu-video-streamer-1  | 2022-07-15 17:44:24.914324: I tensorflow/core/platform/cpu_feature_guard.cc:193] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX512_VNNI
cpu-video-streamer-1  | To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
cpu-video-streamer-1  | INFO:gst_detection_tf:Parameters: {'device': 'CPU', 'preproc_fw': 'cv2', 'data_type': 'FP32', 'onednn': True, 'amx': True, 'inter_op_parallelism': '1', 'intra_op_parallelism': '4', 'database': 'VDMS', 'bounding_box': True, 'face_threshold': 0.7, 'label_file': 'dataset/coco.label', 'ssd_resnet34_fp32_model': 'models/ssd_resnet34_fp32_bs1_pretrained_model.pb', 'ssd_resnet34_bf16_model': 'models/ssd_resnet34_bf16_bs1_pretrained_model.pb', 'ssd_resnet34_int8_model': 'models/ssd_resnet34_int8_bs1_pretrained_model.pb', 'ssd_resnet34_fp16_model': 'models/ssd_resnet34_fp16_bs1_pretrained_model.pb', 'ssd_resnet34_fp32_1200x1200_model': 'models/ssd_resnet34_fp32_1200x1200_pretrained_model.pb', 'ssd_resnet34_int8_1200x1200_model': 'models/ssd_resnet34_int8_1200x1200_pretrained_model.pb', 'ssd_resnet34_bf16_1200x1200_model': 'models/ssd_resnet34_bf16_1200x1200_pretrained_model.pb', 'ssd_resnet34_fp16_gpu_model': 'models/gpu/resnet34_tf.22.1.pb', 'ssd_resnet34_fp32_gpu_model': 'models/gpu/resnet34_tf.22.1.pb'}
cpu-video-streamer-1  | Setting pipeline to PAUSED ...
cpu-video-streamer-1  | Pipeline is PREROLLING ...
cpu-video-streamer-1  | Redistribute latency...
cpu-video-streamer-1  | Redistribute latency...
cpu-video-streamer-1  | 2022-07-15 17:44:25.041257: I tensorflow/compiler/mlir/mlir_graph_optimization_pass.cc:354] MLIR V1 optimization pass is not enabled
cpu-video-streamer-1  | Pipeline is PREROLLED ...
cpu-video-streamer-1  | Setting pipeline to PLAYING ...
cpu-video-streamer-1  | New clock: GstSystemClock
```
...
```
Got EOS from element "pipeline0".
Execution ended after 1:18:36.476950000
Setting pipeline to NULL ...
INFO:gst_detection_tf:Pipeline completes: {'total': 4719.140006303787, 'tf': 4657.993880748749, 'cv': 46.518736600875854, 'np': 1.4765057563781738, 'py': 0.05692410469055176, 'vdms': 0.7926304340362549, 'tf/load_model': 0.4810307025909424, 'np/gst_buf_to_ndarray': 0.5167484283447266, 'cv/normalize': 30.608948469161987, 'cv/resize': 14.067937850952148, 'np/format_image': 0.36579012870788574, 'e2e/preprocess': 56.772953033447266, 'tf/inference': 4657.512850046158, 'np/process_inference_result': 0.5939671993255615, 'py/build_db_data': 0.05692410469055176, 'cv/bound_box': 1.8418502807617188, 'e2e/postprocess': 3.49407958984375, 'vdms/save2db': 0.7926304340362549, 'frames': 8344}
Finished all pipelines
```
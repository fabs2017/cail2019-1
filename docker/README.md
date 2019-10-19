# 开发环境搭建

## build base image
```bash
docker build -t cail:base -f docker/base/Dockerfile .
```

## build dev env
**1.build image**
```bash
docker build -t cail:dev -f docker/dev/Dockerfile .
```
**2.run container**
```bash
BERT_PRETRAINED_MODEL_DIR="/data/resources/bert"
docker run \
    --name cail_dev \
    --restart always \
    --runtime nvidia \
    -v BERT_PRETRAINED_MODEL_DIR:/bert \
    -v /data/padeoe/project/cail:/cail \
    -p 2229:22 \
    -p 2230:8888 \
    -d \
    cail:dev
```

now we can access ssh at localhost:2229 and jupyter notebook at [http://localhost:2230](http://localhost:2230).
 

## build submit image
**1.build image**
```bash
docker build -t cail:submit -f docker/submit/Dockerfile .
```

**2.run test and zip codes**
```bash
docker run \
    --rm \
    -it \
    --runtime nvidia \
    -e LOCAL_USER_ID=`id -u $USER` \
    -e SUBMIT_FILES="main.py model.py model" \
    -v "$PWD":/codes \
    -v "$PWD/data/submit_zip":/output_zip \
    cail:submit
```
It will execute `main.py` and `judger.py` and output the the accuracy of evaluation.
Finally, the compressed codes and models for submit will stored in `$PWD/data/submit_zip`.

## train models
**1.build image**
```bash
docker build -t cail:train -f docker/train/Dockerfile .
```

**2.prepare data**

Put dataset files at `DATA_DIR`, the layout is like this:

```console
$ tree data/
data/
├── raw
│   ├── CAIL2019-SCM-big
│   │   └── SCM_5k.json
│   └── CAIL2019-SCM-small
│       └── input.txt
└── test
    ├── ground_truth.txt
    └── input.txt

3 directories, 3 files
```

**3.train models**
```bash
BERT_PRETRAINED_MODEL="/data/resources/bert/pytorch_chinese_L-12_H-768_A-12/"
OUTPUT_MODELS_DIR="$PWD/model"
DATA_DIR="/data/padeoe/project/cail/data"
docker run \
    --name cail-train \
    --rm \
    -it \
    --runtime nvidia \
    -e LOCAL_USER_ID=`id -u $USER` \
    -v $BERT_PRETRAINED_MODEL:/bert/pytorch_chinese_L-12_H-768_A-12 \
    -v ${DATA_DIR}:/cail/data \
    -v ${OUTPUT_MODELS_DIR}:/cail/model \
    cail:train
```
The model will be saved at `OUTPUT_MODELS_DIR`.

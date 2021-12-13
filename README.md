# 1. Version description
yolov5 version Tags=v2.0, python version 3.7.5

# 2. Prepare the data set

## 2.1 Download the coco2017 data set and decompress it. The decompressed directory is as follows:

```
├── coco_data: #root directory
     ├── train2017 #Training set pictures, about 118287
     ├── val2017 #Validation set pictures, about 5000
     └── annotations # annotation directory
     ├── instances_train2017.json #The training set annotation file corresponding to target detection and segmentation tasks
     ├── instances_val2017.json #Validation set annotation file corresponding to target detection and segmentation tasks
     ├── captions_train2017.json
     ├── captions_val2017.json
     ├── person_keypoints_train2017.json
     └── person_keypoints_val2017.json
```

## 2.2 Generate yolov5 special annotation file

(1) Copy coco/coco2yolo.py and coco/coco_class.txt in the code warehouse to coco_data**root directory**

(2) Run coco2yolo.py

```
python3 coco2yolo.py
```

(3) After running the above script, train2017.txt and val2017.txt will be generated in coco_data **root directory**

# 3. Configure the data set path

Modify the train field and val field in the data/coco.yaml file to point to the train2017.txt and val2017.txt generated in the previous section respectively, such as:

```
train: /data/coco_data/train2017.txt
val: /data/coco_data/val2017.txt
```

# 4.GPU, CPU dependency
Install python dependency package according to requirements-GPU.txt

# 5. NPU dependency
Install the python dependency package according to requirements.txt, and also need to install (NPU-driver.run, NPU-firmware.run, NPU-toolkit.run, torch-ascend.whl, apex.whl)

# 6. Compile and install Opencv-python

In order to get the best image processing performance, ***Please compile and install opencv-python instead of directly installing***. The compilation and installation steps are as follows:

```
export GIT_SSL_NO_VERIFY=true
git clone https://github.com/opencv/opencv.git
cd opencv
mkdir -p build
cd build
cmake -D BUILD_opencv_python3=yes -D BUILD_opencv_python2=no -D PYTHON3_EXECUTABLE=/usr/local/python3.7.5/bin/python3.7m -D PYTHON3_INCLUDE_DIR=/usr/local/python3.7.5/include/python3.7m -D PYTHONARY =/usr/local/python3.7.5/lib/libpython3.7m.so -D PYTHON3_NUMPY_INCLUDE_DIRS=/usr/local/python3.7.5/lib/python3.7/site-packages/numpy/core/include -D PYTHON3_PACKAGES_PATH=/ usr/local/python3.7.5/lib/python3.7/site-packages -D PYTHON3_DEFAULT_EXECUTABLE=/usr/local/python3.7.5/bin/python3.7m ..
make -j$nproc
make install
```

# 7. NPU stand-alone single-card training instruction
bash train_npu_1p.sh

# 8. NPU stand-alone eight-card training instruction
bash train_npu_8p_mp.sh

# 9. NPU evalution instruction
(1) Modify the parameter --coco_instance_path in evaluation_npu_1p.sh to the actual path in the data set. For example, modify the script to

```
python3.7 test.py --data /data/coco.yaml --coco_instance_path /data/coco/annotations/instances_val2017.json --img-size 672 --weight'yolov5_0.pt' --batch-size 32 - device npu --npu 0
```

(2) Start the evaluation

```
bash evaluation_npu_1p.sh
```

# 10. GPU stand-alone single-card training instructions
python train.py --data coco.yaml --cfg yolov5x.yaml --weights'' --batch-size 32 --device 0

# 11.GPU stand-alone eight-card training instruction
python -m torch.distributed.launch --nproc_per_node 8 train.py --data coco.yaml --cfg yolov5x.yaml --weights'' --batch-size 256

# 12.CPU instructions
python train.py --data coco.yaml --cfg yolov5x.yaml --weights'' --batch-size 32 --device cpu

# 13. Export onnx instruction
python export_onnx.py --weights ./xxx.pt --img-size 640 --batch-size 1

# 13. Reults

## 13.1 NPU Traininig Performance

| Model   | Size<br>(pixels) | NPU<br>Nums | Dataset   | Training<br>Data Nums  | Validation<br>Data Nums  | Batch<br>Size | Epochs   | FPS      | Total<br>Training Time<br>(H) |
| :------:| :--------------: | :---------: | :-------: | :--------------------: | :----------------------: | :-----------: | :------: | :------: | :---------------------------: |
| yolo5x  | 640              | 1           | COCO-2017 | 118287                 | 5000                     | 32            | 300      | 51.8     | 188.528                       |
| yolo5x  | 640              | 8           | COCO-2017 | 118287                 | 5000                     | 256           | 300      | 400.5    | 27.287                        |

## 13.2 NPU Inference Performance

| Model     | Size<br>(pixels) | mAP<sup>val<br>0.0 : 1.0 | Speed<br>Atlas 300T<br>(ms)    |
| :-------: | :--------------: | :----------------------: | :----------------------------: |
| yolo5x-1p | 640              | yolo5x-1p                | 48.5                           |
| yolo5x-8p | 640              | yolo5x-8p                | 50.3                           |
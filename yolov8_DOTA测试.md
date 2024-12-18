# 环境搭建：
```PYTHON
conda create -n ultralytics38 python=3.8
conda activate ultralytics38
# 安装torch，本人为cuda11.8
conda install pytorch==2.0.1 torchvision==0.15.2 torchaudio==2.0.2 pytorch-cuda=11.8 -c pytorch -c nvidia
# 安装ultralytics
git clone https://github.com/ultralytics/ultralytics.git
cd ultralytics
pip install -v -e .
```
# 下载DOTAv1数据集

- ultralytics官方给了下载链接：https://github.com/ultralytics/assets/releases/download/v0.0.0/DOTAv1.zip
- 大小为1.9GB，比DOTA官方给的数据集(约20G)小，因为在这里使用的是.jpg格式。图片总数我验证过没有问题。train/val/test图片数量分别为1411/458/937
- 修改ultralytics/cfg/datasets/DOTAv1.yaml文件的path为你存放数据集的路径，之后运行代码它会帮你解压。

新建文件yolo8_obb_train.py，训练代码如下
```PYTHON
from ultralytics import YOLO
# 根据命名会构建对应的模型规模，后缀有obb则构建旋转目标检测模型
model = YOLO("yolov8s-obb.yaml")  # build a new model from YAML
# 单卡训练
results = model.train(data="DOTAv1.yaml", epochs=100, imgsz=1024,)
# 多卡训练，可调batch size和img_size，对于DOTA，我们一般选择1024*1024
# results = model.train(data="DOTAv1.yaml", epochs=100, imgsz=1024, batch=64, device=[0, 1, 2, 3, 4, 5, 6, 7])
```

上述代码跑的是DOTA的整张图片，而DOTA的分辨率非常高，比如`4000*4000`分辨率，将其压缩到`1024*1024`会让检测目标非常小，训练效果不佳。故需要做图片切割。
```PYTHON
from ultralytics.data.split_dota import split_test, split_trainval

# split train and val set, with labels.
split_trainval(
    data_root="path/to/DOTAv1.0/",
    save_dir="path/to/DOTAv1.0-split/",
    rates=[0.5, 1.0, 1.5],  # multiscale
    gap=500,
)
# split test set, without labels.
split_test(
    data_root="path/to/DOTAv1.0/",
    save_dir="path/to/DOTAv1.0-split/",
    rates=[0.5, 1.0, 1.5],  # multiscale
    gap=500,
)
```
复制ultralytics/cfg/datasets路径下的DOTAv1.yaml为DOTAv1_split.yaml，修改里面的path为分割数据集位置。
在切割数据集下进行训练的代码如下，仅修改了model.train后面的data位置：
```PYTHON
from ultralytics import YOLO
model = YOLO("yolov8s-obb.yaml")  # build a new model from YAML
# 单卡训练
results = model.train(data="DOTAv1_split.yaml", epochs=100, imgsz=1024,)
# 多卡训练
# results = model.train(data="DOTAv1_split.yaml", epochs=100, imgsz=1024, batch=64, device=[0, 1, 2, 3, 4, 5, 6, 7])
```
# 如何得到DOTAv1的test测试结果？
test集只有图片没有标注，要得到测试结果需要将你的检测结果上传到DOTA官方，它们会将结果发送到你的邮箱。
官方网址：[DOTA](https://captain-whu.github.io/DOTA/evaluation.html)

## 如何将检测结果打包？
- 新建文件yolov8s_obb_val.py
```PYTHON
from ultralytics import YOLO
# Load a model
model = YOLO("yolov8s-obb.yaml").load("runs/obb/train34/weights/best.pt") # build from YAML and transfer weights
model.val(data="ultralytics/cfg/datasets/DOTAv1_split.yaml",split='test', save_json=True)
# save_json设置为True才会输出检测文件用于上传给DOTA官方
```
在run文件夹下找到对应的predictions_merged_txt文件夹，压缩成.zip格式，上传到DOTA官方即可。

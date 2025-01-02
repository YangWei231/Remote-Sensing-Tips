# 环境配置
参考链接：[AO2-DETR/docs/en/install.md at master · Ixiaohuihuihui/AO2-DETR](https://github.com/Ixiaohuihuihui/AO2-DETR/blob/master/docs/en/install.md)
- cuda:11.3
- torch:1.11.0
```
# cuda
export PATH=/usr/local/cuda-11.3/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.3/lib64:$LD_LIBRARY_PATH
# torch
pip install torch==1.11.0+cu113 torchvision==0.12.0+cu113 torchaudio==0.11.0 --extra-index-url https://download.pytorch.org/whl/cu113

# mmcv-full
pip install mmcv-full==1.5.0 -f https://download.openmmlab.com/mmcv/dist/cu113/torch1.11.0/index.html
# mmdet
pip install mmdet==2.19.0

# AO2
pip install -v -e .
# yapf降版本
pip install yapf==0.40.1
```

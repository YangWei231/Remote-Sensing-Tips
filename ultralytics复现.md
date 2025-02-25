复现时需要获取官方的训练config，获取方式如下：
```python
from ultralytics import YOLO, ASSETS
model = YOLO("yolo12n-obb.pt")
model_cfg=model.ckpt["train_args"]
print(model_cfg)
for key in model_cfg.keys():
    value = model_cfg[key]
    value = f"'{value}'" if type(value)==str else value
    print(f"{key}={value},")
```

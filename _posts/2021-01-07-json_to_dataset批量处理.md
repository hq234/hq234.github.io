---
layout: post
title: win10下json_to_dataset批量处理
categories: [其他笔记]
---

#### 安装labelme（conda内）

```
pip install pyqt5
pip install labelme
```

#### 批处理json_to_dataset  
  
修改D:\anaconda2021\envs\torch_backup\Lib\site-packages\labelme\cli\json_to_dataset.py文件

源码来自互联网，做了简单的修改，将同一类别的文件存放在同一文件夹中

使用方法：`labelme_json_to_dataset.exe D:\json` 

```python
import argparse
import base64
import json
import os
import os.path as osp

import imgviz
import PIL.Image

from labelme.logger import logger
from labelme import utils


def main():
    logger.warning(
        "批量处理json文件。"
        "文件夹中不能有其他文件存在！"
    )
    parser = argparse.ArgumentParser()
    parser.add_argument('json_file')
    parser.add_argument('-o', '--out', default=None)
    args = parser.parse_args()

    json_file = args.json_file

    if args.out is None:
        out_dir = osp.basename(json_file).replace('.', '_')
        out_dir = osp.join(osp.dirname(json_file), out_dir)
    else:
        out_dir = args.out
    if not osp.exists(out_dir):
        os.mkdir(out_dir)

    # 处理多个文件
    count = os.listdir(json_file)
    for i in range(0, len(count)):
        path = os.path.join(json_file, count[i])
        if os.path.isfile(path):
            data = json.load(open(path))
            if data['imageData']:
                imageData = data['imageData']
            else:
                imagePath = os.path.join(os.path.dirname(path), data['imagePath'])
                with open(imagePath, 'rb') as f:
                    imageData = f.read()
                    imageData = base64.b64encode(imageData).decode('utf-8')
            img = utils.img_b64_to_arr(imageData)
            label_name_to_value = {'_background_': 0}
            for shape in sorted(data["shapes"], key=lambda x: x["label"]):
                label_name = shape["label"]
                if label_name in label_name_to_value:
                    label_value = label_name_to_value[label_name]
                else:
                    label_value = len(label_name_to_value)
                    label_name_to_value[label_name] = label_value
            lbl, _ = utils.shapes_to_label(
                img.shape, data["shapes"], label_name_to_value
            )

            label_names = [None] * (max(label_name_to_value.values()) + 1)
            for name, value in label_name_to_value.items():
                label_names[value] = name

            lbl_viz = imgviz.label2rgb(
                label=lbl, img=imgviz.asgray(img), label_names=label_names, loc="rb"
            )

            # 输出路径
            out_dir = osp.basename(count[i]).replace('.json', '')
            out_dir = osp.join(osp.dirname(count[i]), out_dir)

            filename = out_dir
            out_dir = 'Dataset_out'

            if not osp.exists(out_dir):
                os.mkdir(out_dir)
                os.mkdir(osp.join(out_dir, 'image'))
                os.mkdir(osp.join(out_dir, 'label'))
                os.mkdir(osp.join(out_dir, 'label_names'))
                os.mkdir(osp.join(out_dir, 'label_viz'))

            # out_dir = "json_outdir"

            # outdir = 'dataset_output'
            PIL.Image.fromarray(img).save(osp.join(out_dir, 'image', filename+'.png'))
            utils.lblsave(osp.join(out_dir, 'label', filename+'.png'), lbl)
            PIL.Image.fromarray(lbl_viz).save(osp.join(out_dir, 'label_viz', filename+'.png'))

            with open(osp.join(out_dir, 'label_names', filename+'.txt'), 'w') as f:
                for lbl_name in label_names:
                    f.write(lbl_name + '\n')

            logger.info("num:{} Saved to: {}".format(i, out_dir))
```
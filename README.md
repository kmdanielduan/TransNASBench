# TransNAS-Bench-101

This repository contains the publishable code for CVPR 2021 paper [TransNAS-Bench-101: Improving Transferrability and Generalizability of Cross-Task Neural Architecture Search](https://arxiv.org/abs/2105.11871).

In this Markdown file, we show an example how to use TransNAS-Bench-101. The complete network training and evaluation information file can be found through [VEGA](https://www.noahlab.com.hk/opensource/vega/page/doc.html?path=datasets/transnasbench101) or this [Google Drive Folder](https://drive.google.com/drive/folders/1HlLr2ihZX_ZuV3lJX_4i7q4w-ZBdhJ6o?usp=share_link).

## How to use TransNAS-Bench-101

1. Import the API object in `./code/api/api.py` and create an API instance from the `.pth` file in `./api_home/`:
​
```python
from api import TransNASBenchAPI as API
path2nas_bench_file = "./api_home/transnas-bench_v10141024.pth"
api = API(path2nas_bench_file)
```

2. Check the task information, number of architectures evaluated, and search spaces:

```python
# show number of architectures and number of tasks
length = len(api)
task_list = api.task_list # list of tasks
print(f"This API contains {length} architectures in total across {len(task_list)} tasks.")
# This API contains 7352 architectures in total across 7 tasks.

# Check all model encoding
search_spaces = api.search_spaces # list of search space names
all_arch_dict = api.all_arch_dict # {search_space : list_of_architecture_names}
for ss in search_spaces:
   print(f"Search space '{ss}' contains {len(all_arch_dict[ss])} architectures.")
print(f"Names of 7 tasks: {task_list}")
# Search space 'macro' contains 3256 architectures.
# Search space 'micro' contains 4096 architectures.
# Names of 7 tasks: ['class_scene', 'class_object', 'room_layout', 'jigsaw', 'segmentsemantic', 'normal', 'autoencoder']
```

3. Since different tasks may require different evaluation metrics, hence `metric_dict` showing the used metrics can be retrieved from `api.metrics_dict`. TransNAS-Bench API also recorded the model inference time, backbone/model parameters, backbone/model FLOPs in `api.infor_names`.

```python
metrics_dict = api.metrics_dict # {task_name : list_of_metrics}
info_names = api.info_names # list of model info names

# check the training information of the example task
task = "class_object"
print(f"Task {task} recorded the following metrics: {metrics_dict[task]}")
print(f"The following model information are also recorded: {info_names}")
# Task class_object recorded the following metrics: ['train_top1', 'train_top5', 'train_loss', 'valid_top1', 'valid_top5', 'valid_loss', 'test_top1', 'test_top5', 'test_loss', 'time_elapsed']
# The following model information are also recorded: ['inference_time', 'encoder_params', 'model_params', 'model_FLOPs', 'encoder_FLOPs']
```

4. Query the results of an architecture by arch string
​
```python
# Given arch string
xarch = api.index2arch(1) # '64-2311-basic'
for xtask in api.task_list:
    print(f'----- {xtask} -----')
    print(f'--- info ---')
    for xinfo in api.info_names:
        print(f"{xinfo} : {api.get_model_info(xarch, xtask, xinfo)}")
    print(f'--- metrics ---')
    for xmetric in api.metrics_dict[xtask]:
        print(f"{xmetric} : {api.get_single_metric(xarch, xtask, xmetric, mode='best')}")
        print(f"best epoch : {api.get_best_epoch_status(xarch, xtask, metric=xmetric)}")
        print(f"final epoch : {api.get_epoch_status(xarch, xtask, epoch=-1)}")
        if ('valid' in xmetric and 'loss' not in xmetric) or ('valid' in xmetric and 'neg_loss' in xmetric):
            print(f"\nbest_arch -- {xmetric}: {api.get_best_archs(xtask, xmetric, 'micro')[0]}")
```

A complete example is given in `code/api/example.py`
- `cd code/api`
- `python example.py`

## Example network encoding in both search spaces

```markdown
Macro example network: 64-1234-basic
- Base channel: 64
- Macro skeleton: 1234 (4 stacked modules)
  - [m1(normal)-m2(channelx2)-m3(resolution/2)-m4(channelx2 & resolution/2)]
- Cell structure: basic (ResNet Basic Block)

Micro example network: 64-41414-1_02_333
- Base channel: 64
- Macro skeleton: 41414 (5 stacked modules)
  - [m1(channelx2 & resolution/2)-m2(normal)-m3(channelx2 & resolution/2)-m4(normal)-m5(channelx2 & resolution/2)]
- Cell structure: 1_02_333 (4 nodes, 6 edges)
  - node0: input tensor
  - node1: Skip-Connect( node0 ) # 1
  - node2: None( node0 ) + Conv1x1( node1 ) # 02
  - node3: Conv3x3( node0 ) + Conv3x3( node1 ) + Conv3x3( node2 ) # 333
```

# Citation

If you find that TransNAS-Bench-101 helps your research, please consider citing it:

```bibtex
@inproceedings{duan2021transnas,
  title = {TransNAS-Bench-101: Improving Transferability and Generalizability of Cross-Task Neural Architecture Search},
  author = {Duan, Yawen and Chen, Xin and Xu, Hang and Chen, Zewei and Liang, Xiaodan and Zhang, Tong and Li, Zhenguo},
  booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages = {5251--5260},
  year = {2021}
}
```

# License
MIT license.

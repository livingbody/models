# 声音分类

声音分类和检测是声音算法的一个热门研究方向。  
对于声音分类任务，传统机器学习的一个常用做法是首先人工提取音频的时域和频域的多种特征并做特征选择、组合、变换等，然后基于SVM或决策树进行分类。而端到端的深度学习则通常利用深度网络如RNN，CNN等直接对声间波形(waveform)或时频特征(time-frequency)进行特征学习(representation learning)和分类预测。

在IEEE ICASSP 2017 大会上，谷歌开放了一个大规模的音频数据集[Audioset](https://research.google.com/audioset/)。该数据集包含了 632 类的音频类别以及 2,084,320 条人工标记的每段 10 秒长度的声音剪辑片段（来源于YouTube视频）。目前该数据集已经有210万个已标注的视频数据，5800小时的音频数据，经过标记的声音样本的标签类别为527。

`PANNs`([PANNs: Large-Scale Pretrained Audio Neural Networks for Audio Pattern Recognition](https://arxiv.org/pdf/1912.10211.pdf))是基于Audioset数据集训练的声音分类/识别的模型。经过预训练后，模型可以用于提取音频的embbedding。本示例将使用`PANNs`的预训练模型Finetune完成声音分类的任务。


## 模型简介

PaddleAudio提供了PANNs的CNN14、CNN10和CNN6的预训练模型，可供用户选择使用：
- CNN14: 该模型主要包含12个卷积层和2个全连接层，模型参数的数量为79.6M，embbedding维度是2048。
- CNN10: 该模型主要包含8个卷积层和2个全连接层，模型参数的数量为4.9M，embbedding维度是512。
- CNN6: 该模型主要包含4个卷积层和2个全连接层，模型参数的数量为4.5M，embbedding维度是512。


## 快速开始

### 模型训练

以环境声音分类数据集`ESC50`为示例，运行下面的命令，可在训练集上进行模型的finetune，支持单机的单卡训练和多卡分布式训练。关于如何使用`paddle.distributed.launch`启动分布式训练，请查看[PaddlePaddle2.0分布式训练](https://www.paddlepaddle.org.cn/documentation/docs/zh/tutorial/quick_start/high_level_api/high_level_api.html#danjiduoka)。

```shell
$ unset CUDA_VISIBLE_DEVICES
$ python -m paddle.distributed.launch --gpus "0" train.py --device gpu --epochs 50 --batch_size 16 --num_worker 16 --checkpoint_dir ./checkpoint --save_freq 10
```

可支持配置的参数：

- `device`: 选用什么设备进行训练，可选cpu或gpu，默认为gpu。如使用gpu训练则参数gpus指定GPU卡号。
- `epochs`: 训练轮次，默认为50。
- `learning_rate`: Fine-tune的学习率；默认为5e-5。
- `batch_size`: 批处理大小，请结合显存情况进行调整，若出现显存不足，请适当调低这一参数；默认为16。
- `num_workers`: Dataloader获取数据的子进程数。默认为0，加载数据的流程在主进程执行。
- `checkpoint_dir`: 模型参数文件和optimizer参数文件的保存目录，默认为`./checkpoint`。
- `save_freq`: 训练过程中的模型保存频率，默认为10。
- `log_freq`: 训练过程中的信息打印频率，默认为10。

示例代码中使用的预训练模型为`CNN14`，如果想更换为其他预训练模型，可通过以下方式执行：
```python
from model import SoundClassifier
from paddleaudio.datasets import ESC50
from paddleaudio.models.panns import cnn14, cnn10, cnn6

# CNN14
backbone = cnn14(pretrained=True, extract_embedding=True)
model = SoundClassifier(backbone, num_class=len(ESC50.label_list))

# CNN10
backbone = cnn10(pretrained=True, extract_embedding=True)
model = SoundClassifier(backbone, num_class=len(ESC50.label_list))

# CNN6
backbone = cnn6(pretrained=True, extract_embedding=True)
model = SoundClassifier(backbone, num_class=len(ESC50.label_list))
```

### 模型预测

```shell
export CUDA_VISIBLE_DEVICES=0
python -u predict.py --device gpu --wav ./dog.wav --top_k 3 --checkpoint ./checkpoint/epoch_50/model.pdparams
```

可支持配置的参数：
- `device`: 选用什么设备进行训练，可选cpu或gpu，默认为gpu。如使用gpu训练则参数gpus指定GPU卡号。
- `wav`: 指定预测的音频文件。
- `top_k`: 预测显示的top k标签的得分，默认为1。
- `checkpoint`: 模型参数checkpoint文件。

输出的预测结果如下：
```
[/audio/dog.wav]
Dog: 0.9999538660049438
Clock tick: 1.3341237718123011e-05
Cat: 6.579841738130199e-06
```

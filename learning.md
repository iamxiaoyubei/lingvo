# Lingvo代码学习
## 代码结构
根目录下trainer.py为lingvo的入口，tools目录下包含着数据库处理的工具函数，core目录包含着基础网络（父类）的核心代码，tasks目录包含着各个不同任务的代码。


其中我们使用到的asr任务下，params中的librispeech.py定义了两个语音识别参数模型：Librispeech960Grapheme和Librispeech960Wpm，在asr根目录下，model.py是asr任务的网络模型，input_generator.py是asr任务的网络输入处理部分，其他是网络的各个模块部分。这各种模型与core目录下的父类模型都是各自对应的继承关系。


asr目录下的tools是下载和处理数据集数据的script和计算WER的一个单独模块。

## 数据集处理
librispeech数据集为一个多说话人语音识别数据集，我们使用到的训练集包含train-clean-100,train-clean-360,train-other-500，使用到的测试数据集包含dev-clean,dev-other,test-clean,test-other。


clean是指包含的是“clean”音频的数据集，other是指可能并不“clean”的音频数据集，更具有挑战性。


每一个下载下来的数据集（.tar.gz）结构如下：
```
<corpus root>
    |
    .- README.TXT
    |
    .- READERS.TXT
    |
    .- CHAPTERS.TXT
    |
    .- BOOKS.TXT
    |
    .- train-clean-100/
                   |
                   .- 19/
                       |
                       .- 198/
                       |    |
                       |    .- 19-198.trans.txt
                       |    |    
                       |    .- 19-198-0001.flac
                       |    |
                       |    .- 19-198-0002.flac
                       |    |
                       |    ...
                       |
                       .- 227/
                            | ...



```
我们需要使用到的是***.trans.txt文件，如19-198.trans.txt，其中包含着ID19的people读ID198的chapter的语音文本(transcript)，和***.flac文件，如19-198-0001.flac其中包含着ID19的people读ID198的chapter的第一句语音。


在数据集处理中，我们首先是将所有训练数据集的transcripts汇总成一个文件，其次将所有训练数据集的每一条utterance语音处理成log Mel,再将(utterance id, log mel, utterance transcript)作为一个feature，将所有这样的feature存入***.tfrecords-**-of-**文件中，从而生成训练数据集，总共有281241条utterance，训练数据大小约为110.6GB。


同理，测试数据集也是按照这样处理成feature保存，总共有11126条utterance，测试数据大小约为2.4GB。


相关处理代码见/lingvo/tasks/asr/tools/librispeech.** .parameterize **.sh和/lingvo/tools/create_asr_features.py。


## 训练参数
参数从librispeech.py开始追溯修改。
未完待续...

## Reference
- https://tensorflow.github.io/lingvo/index.html
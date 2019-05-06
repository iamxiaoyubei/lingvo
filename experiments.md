# Lingvo实验进展
## 环境配置
使用docker进行环境配置，首先需要安装好Docker CE以及nvidia-docker2（不安装nvidia-docker2会报错`docker: Error response from daemon: Unknown runtime specified nvidia`）:
- https://docs.docker.com/install/linux/docker-ce/ubuntu/
- https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)

之后，按照README说明配置docker:
```
LINGVO_DIR="/data/xiaoyubei/codes/lingvo"
LINGVO_DEVICE="gpu"
sudo docker build --tag tensorflow:lingvo $(test "$LINGVO_DEVICE" = "gpu" && echo "--build-arg base_image=nvidia/cuda:10.0-cudnn7-runtime-ubuntu16.04") - < ${LINGVO_DIR}/docker/dev.dockerfile
```
成功后会安装好docker环境，出现如下信息：
```
Sending build context to Docker daemon   5.12kB
Step 1/19 : ARG cpu_base_image="ubuntu:16.04"
Step 2/19 : ARG base_image=$cpu_base_image
Step 3/19 : FROM $base_image
10.0-cudnn7-runtime-ubuntu16.04: Pulling from nvidia/cuda
34667c7e4631: Pull complete 
d18d76a881a4: Pull complete 
119c7358fbfc: Pull complete 
2aaf13f3eff0: Pull complete 
643564d518c8: Pull complete 
1fea03e629a4: Pull complete 
45402f4cf61d: Pull complete 
86f75b2a221d: Pull complete 
9e547bd511ba: Pull complete 
Digest: sha256:86723af6ed8a39f9a10da494ca01ead91e9f28c1d09f73685c4e19befc6f9c50
Status: Downloaded newer image for nvidia/cuda:10.0-cudnn7-runtime-ubuntu16.04
 ---> e8924aa8628f
Step 4/19 : LABEL maintainer="Patrick Nguyen <drpng@google.com>"
 ---> Running in 8934c2d10c0e
Removing intermediate container 8934c2d10c0e
 ---> 1292f23168db
Step 5/19 : ARG cpu_base_image="ubuntu:16.04"
 ---> Running in 98dea27e7a70
Removing intermediate container 98dea27e7a70
 ---> 7e35952c4b4f
Step 6/19 : ARG base_image=$cpu_base_image
 ---> Running in 87d4ce0d3c03
Removing intermediate container 87d4ce0d3c03
 ---> 997fe7260941

...

## Build informations
   - [Commit](https://github.com/bazelbuild/bazel/commit/76d175c)
Uncompressing.......

Bazel is now installed!

Make sure you have "/usr/local/bin" in your path. You can also activate bash
completion by adding the following line to your ~/.bashrc:
  source /usr/local/lib/bazel/bin/bazel-complete.bash

See http://bazel.build/docs/getting-started.html to start a new project!
Removing intermediate container 8c3648e0f056
 ---> 56fc0968ce36
Step 16/19 : EXPOSE 6006
 ---> Running in 1c5cf5a442a1
Removing intermediate container 1c5cf5a442a1
 ---> 113eb6841e25
Step 17/19 : EXPOSE 8888
 ---> Running in 040503b61f78
Removing intermediate container 040503b61f78
 ---> 0dd1ca05d72b
Step 18/19 : WORKDIR "/tmp/lingvo"
 ---> Running in 645bdb076244
Removing intermediate container 645bdb076244
 ---> 0d0a3ca83eae
Step 19/19 : CMD ["/bin/bash"]
 ---> Running in 7faac830ecb4
Removing intermediate container 7faac830ecb4
 ---> 479d60ec6ee1
Successfully built 479d60ec6ee1
Successfully tagged tensorflow:lingvo

```

其中，中途出现了报错，没有管它，目前还没遇到由这个影响到的问题：
```
debconf: delaying package configuration, since apt-utils is not installed
...
Enabling: jupyter_http_over_ws
- Writing config: /root/.jupyter
    - Validating...
      jupyter_http_over_ws 0.0.6 OK
...
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7.
```
接着run刚刚build的docker环境：
```
sudo docker run --rm $(test "$LINGVO_DEVICE" = "gpu" && echo "--runtime=nvidia") -it -v ${LINGVO_DIR}:/tmp/lingvo -v ${HOME}/.gitconfig:/home/${USER}/.gitconfig:ro -p 6006:6006 -p 8888:8888 --name lingvo tensorflow:lingvo bash
```
则会进入该环境：
```
(base) dm@dm-System-Product-Name:~/Documents/codes/lingvo$ sudo docker run --rm $(test "$LINGVO_DEVICE" = "gpu" && echo "--runtime=nvidia") -it -v ${LINGVO_DIR}:/tmp/lingvo -v ${HOME}/.gitconfig:/home/${USER}/.gitconfig:ro -p 6006:6006 -p 8888:8888 --name lingvo tensorflow:lingvo bash
[sudo] password for dm: 
root@8e094fbc50c3:/tmp/lingvo#
```
在该环境中测试：
```
root@8e094fbc50c3:/tmp/lingvo# bazel test -c opt //lingvo:trainer_test //lingvo:models_test
Extracting Bazel installation...
Starting local Bazel server and connecting to it...
INFO: Repository rule 'subpar' returned: {"remote": "https://github.com/google/subpar", "commit": "07ff5feb7c7b113eea593eb6ec50b51099cf0261", "shallow_since": "2018-04-26", "init_submodules": False, "verbose": False, "strip_prefix": "", "patches": [], "patch_tool": "patch", "patch_args": ["-p0"], "patch_cmds": [], "name": "subpar"}
INFO: Analysed 2 targets (40 packages loaded).
INFO: Found 2 test targets...
FAIL: //lingvo:trainer_test (shard 5 of 5) (see /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_5_of_5/test.log)
FAIL: //lingvo:trainer_test (shard 3 of 5) (see /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_3_of_5/test.log)
FAIL: //lingvo:trainer_test (shard 4 of 5) (see /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_4_of_5/test.log)

FAILED: //lingvo:trainer_test (Summary)
      /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_5_of_5/test.log
      /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_3_of_5/test.log
      /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_4_of_5/test.log
INFO: Elapsed time: 107.898s, Critical Path: 73.73s
INFO: 29 processes: 29 processwrapper-sandbox.
INFO: Build completed, 1 test FAILED, 45 total actions
//lingvo:models_test                                                     PASSED in 10.6s
//lingvo:trainer_test                                                    FAILED in 3 out of 5 in 64.1s
  Stats over 5 runs: max = 64.1s, min = 4.4s, avg = 18.5s, dev = 23.2s
  /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_5_of_5/test.log
  /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_3_of_5/test.log
  /root/.cache/bazel/_bazel_root/17eb95f0bc03547f4f1319e61997e114/execroot/__main__/bazel-out/k8-opt/testlogs/lingvo/trainer_test/shard_4_of_5/test.log

Executed 2 out of 2 tests: 1 test passes and 1 fails locally.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command linINFO: Build completed, 1 test FAILED, 45 total actions
root@8e094fbc50c3:/tmp/lingvo# 
```
## librispeech数据集下载与处理
### 下载数据集
代码中给出了下载和处理librispeech的scripts在./lingvo/tasks/asr/tools/中，首先要下载aria2下载文件工具：
```
sudo apt-get install aria2
```
于克隆的lingvo根目录下run scripts:
```
sh ./lingvo/tasks/asr/tools/librispeech.01.download_train.sh
```
便开始了下载：
```
 *** Download Progress Summary as of Wed Apr 24 15:25:23 2019 ***
======================================================================================================
[#7c4ac8 18MiB/5.9GiB(0%) CN:16 DL:304KiB ETA:5h40m20s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-clean-100.tar.gz
------------------------------------------------------------------------------------------------------
[#7f1e27 24MiB/21GiB(0%) CN:16 DL:378KiB ETA:16h30m33s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-clean-360.tar.gz
------------------------------------------------------------------------------------------------------
[#ca61a0 20MiB/28GiB(0%) CN:16 DL:375KiB ETA:22h4m26s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-other-500.tar.gz
------------------------------------------------------------------------------------------------------
```
需等待许久，同时下载devtest数据:
```
sh ./lingvo/tasks/asr/tools/librispeech.02.download_devtest.sh
```
实测用迅雷下载更快

### 处理数据集
记得挂载数据集的位置到docker的相应位置（/tmp/librispeech），否则是无法通过本机绝对路径找到数据集的。其次记得bazel build相应的处理文件（create_asr_features），可以CTRL+F寻找相应文件的bazel name
```
LIBRISPEECH_DIR="/data/xiaoyubei/datasets/librispeech"
sudo docker run --rm $(test "$LINGVO_DEVICE" = "gpu" && echo "--runtime=nvidia") -it -v ${LINGVO_DIR}:/tmp/lingvo -v ${LIBRISPEECH_DIR}:/tmp/librispeech -v ${HOME}/.gitconfig:/home/${USER}/.gitconfig:ro -p 6006:6006 -p 8888:8888 --name lingvo tensorflow:lingvo bash

bazel build -c opt //lingvo/tools:create_asr_features
sh ./lingvo/tasks/asr/tools/librispeech.03.parameterize_train.sh
... takes about 2h ...
sh ./lingvo/tasks/asr/tools/librispeech.04.parameterize_devtest.sh
... takes about 30min ...
```
处理结束会生成110.6GB的train文件和2.4GB的devtest文件


## 训练模型
```
root@d4bff1951ef0:/tmp/lingvo# bazel build -c opt //lingvo:trainer
Starting local Bazel server and connecting to it...
INFO: Analysed target //lingvo:trainer (37 packages loaded, 4708 targets configured).
INFO: Found 1 target...
Target //lingvo:trainer up-to-date:
  bazel-bin/lingvo/trainer
INFO: Elapsed time: 4.297s, Critical Path: 0.19s
INFO: 1 process: 1 processwrapper-sandbox.
INFO: Build completed successfully, 5 total actions
root@d4bff1951ef0:/tmp/lingvo# 
```
目前出现了Memcpy failed/CUDNN_STATUS_INTERNAL_ERROR情况：
```
bazel-bin/lingvo/trainer --run_locally=gpu --mode=sync --model=asr.librispeech.Librispeech960Grapheme --logdir=/tmp/librispeech/log --logtostderr --enable_asserts=false

I0425 13:03:42.883239 139972916016896 trainer.py:270] Save checkpoint done: /tmp/librispeech/log/train/ckpt-00000000
2019-04-25 13:03:42.920659: I tensorflow/stream_executor/stream.cc:1852] [stream=0x77b6df0,impl=0x77b6e90] did not wait for [stream=0x77b6750,impl=0x77b67f0]
2019-04-25 13:03:42.927286: I tensorflow/stream_executor/stream.cc:1852] [stream=0x77b6df0,impl=0x77b6e90] did not wait for [stream=0x77b6750,impl=0x77b67f0]
2019-04-25 13:03:42.927843: I tensorflow/stream_executor/stream.cc:4800] [stream=0x77b6df0,impl=0x77b6e90] did not memcpy host-to-device; source: 0x7f4e2ce4ecc0
2019-04-25 13:03:42.927929: F tensorflow/core/common_runtime/gpu/gpu_util.cc:339] CPU->GPU Memcpy failed
Aborted (core dumped)
...
2019-04-26 02:29:47.176376: I lingvo/core/ops/record_yielder.cc:341] Epoch 1 /tmp/librispeech/train/train.tfrecords-*
2019-04-26 02:30:42.692324: I tensorflow/stream_executor/platform/default/dso_loader.cc:43] Successfully opened dynamic library libcudnn.so.7
2019-04-26 02:30:51.931265: E tensorflow/stream_executor/cuda/cuda_dnn.cc:329] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
2019-04-26 02:30:51.970948: W ./tensorflow/stream_executor/stream.h:1988] attempting to perform DNN operation using StreamExecutor without DNN support
2019-04-26 02:30:52.362986: E tensorflow/stream_executor/cuda/cuda_dnn.cc:329] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
2019-04-26 02:30:52.494345: E tensorflow/stream_executor/cuda/cuda_dnn.cc:329] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR
2019-04-26 02:31:34.607178: I lingvo/core/ops/record_yielder.cc:313] 0x7fb16bfdd060Basic record yielder exit
I0426 02:31:35.971426 140410399749888 base_runner.py:236] trainer done (fatal error).
I0426 02:31:35.982585 140410399749888 base_runner.py:115] trainer exception: cudnn PoolForward launch failed
	 [[node fprop/librispeech/tower_0_0/enc/conv_L0/max_pool (defined at tmp/lingvo/bazel-bin/lingvo/trainer.runfiles/__main__/lingvo/core/conv_layers_with_time_padding.py:95) ]]
	 [[add_31_G438]]
```
尝试 https://github.com/tensorflow/tensorflow/issues/24496 方式，无法解决，个人觉得是显存不够，打算把实验迁移到服务器上进行。

服务器上无需sudo run docker的方式：http://www.docker.org.cn/book/install/run-docker-without-sudo-30.html ， 之后就可以无需sudo权限来run实验。

目前已将环境和数据迁移到服务器上，重新用上面环境配置命令配置好之后，用下面的命令run实验，其中若忘记设置挂载LINGVO_DIR则会出现bazel build无法成功需要in workpace build的问题。若忘记设置挂载LIBRISPEECH_DIR则会找不到数据集。
```
LINGVO_DIR="/home/gongke/xiaoyubei/codes/lingvo"
LINGVO_DEVICE="gpu"
LIBRISPEECH_DIR="/home/gongke/xiaoyubei/datasets/librispeech"
docker run --rm $(test "$LINGVO_DEVICE" = "gpu" && echo "--runtime=nvidia") -it -v ${LINGVO_DIR}:/tmp/lingvo -v ${LIBRISPEECH_DIR}:/tmp/librispeech -v ${HOME}/.gitconfig:/home/${USER}/.gitconfig:ro -p 6006:6006 -p 8888:8888 --name lingvo tensorflow:lingvo bash
bazel build -c opt //lingvo:trainer
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 bazel-bin/lingvo/trainer --run_locally=gpu --mode=sync --model=asr.librispeech.Librispeech960Wpm --logdir=/tmp/librispeech/log --logtostderr --enable_asserts=false
```

需要修改 librispeech.py line 65的batch size，否则太大还是run不起来。成功run起来结果如图：
```
I0428 02:37:36.455261 139886229165824 trainer.py:371] Steps/second: 0.000000, Examples/second: 0.000000
I0428 02:37:46.467176 139886229165824 trainer.py:371] Steps/second: 0.000000, Examples/second: 0.000000
I0428 02:37:47.125787 139886136911616 base_runner.py:115] step:     1 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:398.85916 grad_scale_all:0 log_pplx:9.8754921 log_pplx/logits:9.8754921 loss:9.8754921 loss/logits:9.8754921 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:37:55.583883 139886136911616 trainer.py:520] step:     2 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:379.92902 grad_scale_all:0 log_pplx:10.033342 log_pplx/logits:10.033342 loss:10.033342 loss/logits:10.033342 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:37:56.473912 139886229165824 trainer.py:371] Steps/second: 0.199862, Examples/second: 0.399724
I0428 02:37:56.474525 139886229165824 trainer.py:275] Write summary @2
I0428 02:38:02.778817 139886136911616 trainer.py:520] step:     3 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:434.77341 grad_scale_all:0 log_pplx:9.9486217 log_pplx/logits:9.9486217 loss:9.9486217 loss/logits:9.9486217 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:38:10.370831 139886136911616 trainer.py:520] step:     4 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:354.99301 grad_scale_all:0 log_pplx:9.9383869 log_pplx/logits:9.9383869 loss:9.9383869 loss/logits:9.9383869 num_samples_in_batch:2 var_norm/all:1088.4357
2019-04-28 02:38:11.550715: I ./lingvo/core/ops/input_common.h:68] Create RecordProcessor
2019-04-28 02:38:11.661113: I lingvo/core/ops/input_common.cc:30] Input source weights are empty, fall back to legacy behavior.
2019-04-28 02:38:11.662980: I lingvo/core/ops/record_yielder.cc:288] 0x7f35bfadfa70 Record yielder start
2019-04-28 02:38:11.663024: I lingvo/core/ops/record_yielder.cc:290] Randomly seed RecordYielder.
2019-04-28 02:38:11.663063: I ./lingvo/core/ops/input_common.h:73] Create batcher
2019-04-28 02:38:11.663112: I lingvo/core/ops/record_yielder.cc:341] Epoch 1 /tmp/librispeech/train/train.tfrecords-*
I0428 02:38:20.550335 139886136911616 trainer.py:520] step:     5 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:303.15164 grad_scale_all:0 log_pplx:9.7810946 log_pplx/logits:9.7810946 loss:9.7810946 loss/logits:9.7810946 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:38:26.903563 139886136911616 trainer.py:520] step:     6 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:463.33191 grad_scale_all:0 log_pplx:9.9937429 log_pplx/logits:9.9937429 loss:9.9937429 loss/logits:9.9937429 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:38:38.987941 139886136911616 trainer.py:520] step:     7 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:374.92935 grad_scale_all:0 log_pplx:10.035495 log_pplx/logits:10.035495 loss:10.035495 loss/logits:10.035495 num_samples_in_batch:2 var_norm/all:1088.4357
2019-04-28 02:38:39.017985: I lingvo/core/ops/record_batcher.cc:344] 68 total seconds passed. Total records yielded: 28. Total records skipped: 0
I0428 02:38:48.481496 139886229165824 trainer.py:284] Write summary done: step 2
I0428 02:38:48.488065 139886229165824 base_runner.py:115] step:     2, steps/sec: 0.20, examples/sec: 0.40
I0428 02:38:48.493494 139886229165824 trainer.py:371] Steps/second: 0.112855, Examples/second: 0.225710
I0428 02:38:49.164124 139886136911616 trainer.py:520] step:     8 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:309.8277 grad_scale_all:0 log_pplx:10.011168 log_pplx/logits:10.011168 loss:10.011168 loss/logits:10.011168 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:38:58.354603 139886136911616 trainer.py:520] step:     9 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:332.31821 grad_scale_all:0 log_pplx:10.066358 log_pplx/logits:10.066358 loss:10.066358 loss/logits:10.066358 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:38:58.535938 139886229165824 trainer.py:371] Steps/second: 0.124880, Examples/second: 0.222010
I0428 02:39:06.659197 139886136911616 trainer.py:520] step:    10 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:366.88666 grad_scale_all:0 log_pplx:9.7860727 log_pplx/logits:9.7860727 loss:9.7860727 loss/logits:9.7860727 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:08.515033 139886229165824 trainer.py:371] Steps/second: 0.121880, Examples/second: 0.243760
I0428 02:39:11.839998 139886136911616 trainer.py:520] step:    11 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:550.30212 grad_scale_all:0 log_pplx:9.8040724 log_pplx/logits:9.8040724 loss:9.8040724 loss/logits:9.8040724 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:18.289273 139886136911616 trainer.py:520] step:    12 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:325.69394 grad_scale_all:0 log_pplx:10.02005 log_pplx/logits:10.02005 loss:10.02005 loss/logits:10.02005 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:18.565817 139886229165824 trainer.py:371] Steps/second: 0.130295, Examples/second: 0.238874
I0428 02:39:23.229195 139886136911616 trainer.py:520] step:    13 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:387.82568 grad_scale_all:0 log_pplx:10.063722 log_pplx/logits:10.063722 loss:10.063722 loss/logits:10.063722 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:28.536034 139886229165824 trainer.py:371] Steps/second: 0.127365, Examples/second: 0.254730
I0428 02:39:32.638768 139886136911616 trainer.py:520] step:    14 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:381.58237 grad_scale_all:0 log_pplx:9.8603516 log_pplx/logits:9.8603516 loss:9.8603516 loss/logits:9.8603516 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:38.549165 139886229165824 trainer.py:371] Steps/second: 0.124909, Examples/second: 0.249817
I0428 02:39:41.181643 139886136911616 trainer.py:520] step:    15 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:304.54611 grad_scale_all:0 log_pplx:9.9584513 log_pplx/logits:9.9584513 loss:9.9584513 loss/logits:9.9584513 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:48.559518 139886229165824 trainer.py:371] Steps/second: 0.122858, Examples/second: 0.245716
I0428 02:39:50.634742 139886136911616 trainer.py:520] step:    16 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:364.39774 grad_scale_all:0 log_pplx:10.009266 log_pplx/logits:10.009266 loss:10.009266 loss/logits:10.009266 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:39:58.568286 139886229165824 trainer.py:371] Steps/second: 0.121119, Examples/second: 0.242239
I0428 02:39:59.991051 139886136911616 trainer.py:520] step:    17 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:363.9465 grad_scale_all:0 log_pplx:9.9929256 log_pplx/logits:9.9929256 loss:9.9929256 loss/logits:9.9929256 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:40:08.580627 139886229165824 trainer.py:371] Steps/second: 0.119623, Examples/second: 0.239245
I0428 02:40:08.798882 139886136911616 trainer.py:520] step:    18 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:422.88327 grad_scale_all:0 log_pplx:9.7943659 log_pplx/logits:9.7943659 loss:9.7943659 loss/logits:9.7943659 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:40:17.796449 139886136911616 trainer.py:520] step:    19 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:522.08038 grad_scale_all:0 log_pplx:9.9491949 log_pplx/logits:9.9491949 loss:9.9491949 loss/logits:9.9491949 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:40:18.584604 139886229165824 trainer.py:371] Steps/second: 0.124903, Examples/second: 0.249807
I0428 02:40:26.315630 139886136911616 trainer.py:520] step:    20 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:281.60037 grad_scale_all:0 log_pplx:9.8712187 log_pplx/logits:9.8712187 loss:9.8712187 loss/logits:9.8712187 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:40:28.596860 139886229165824 trainer.py:371] Steps/second: 0.123358, Examples/second: 0.259052
I0428 02:40:30.005242 139886136911616 trainer.py:520] step:    21 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:528.17944 grad_scale_all:0 log_pplx:10.01756 log_pplx/logits:10.01756 loss:10.01756 loss/logits:10.01756 num_samples_in_batch:4 var_norm/all:1088.4357
I0428 02:40:38.083539 139886136911616 trainer.py:520] step:    22 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:306.41714 grad_scale_all:0 log_pplx:9.9797964 log_pplx/logits:9.9797964 loss:9.9797964 loss/logits:9.9797964 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:40:38.605829 139886229165824 trainer.py:371] Steps/second: 0.127804, Examples/second: 0.267226
I0428 02:40:48.617949 139886229165824 trainer.py:371] Steps/second: 0.120779, Examples/second: 0.252538
I0428 02:40:48.996296 139886136911616 trainer.py:520] step:    23 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:360.05887 grad_scale_all:0 log_pplx:9.9907274 log_pplx/logits:9.9907274 loss:9.9907274 loss/logits:9.9907274 num_samples_in_batch:2 var_norm/all:1088.4357
2019-04-28 02:40:49.010331: I lingvo/core/ops/record_batcher.cc:344] 198 total seconds passed. Total records yielded: 61. Total records skipped: 0
I0428 02:40:54.464982 139886136911616 trainer.py:520] step:    24 fraction_of_correct_next_step_preds:0 fraction_of_correct_next_step_preds/logits:0 grad_norm/all:418.73044 grad_scale_all:0 log_pplx:9.9715481 log_pplx/logits:9.9715481 loss:9.9715481 loss/logits:9.9715481 num_samples_in_batch:2 var_norm/all:1088.4357
I0428 02:40:58.626229 139886229165824 trainer.py:371] Steps/second: 0.124896, Examples/second: 0.260201
```

出现并没有利用到多张卡，而仅仅只在一张卡中跑的情况，目前加上参数--worker_gpus=8即可解决：
To use GPU, add `--config=cuda` to build command and set `--run_locally=gpu`.
```
bazel build -c opt //lingvo:trainer --config=cuda
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 bazel-bin/lingvo/trainer --run_locally=gpu --mode=sync --model=asr.librispeech.Librispeech960Wpm --logdir=/tmp/librispeech/log --logtostderr --enable_asserts=false --worker_gpus=8
```

### tensorboard查看训练情况
服务器无法访问tensorboard开启的网站，出现unreachable问题，需要将events文件传到本地查看：
```
scp gongke@192.168.68.51:/home/gongke/xiaoyubei/datasets/librispeech/log/train/events.* /home/dm/Documents/tmp/
tensorboard --logdir .
```

### 训练出现的问题
在20Kstep时，loss本来训练到0.06左右，突然上升至1，通过查阅发现是variational noise的问题，如下issue：
https://github.com/tensorflow/lingvo/issues/13

将librispeech.py line 173中的vn_standard降低一点尝试改善

### Model Inference
在lingvo根目录下：
```
bazel run -c opt //lingvo:ipython_kernel
```
或者将librispeech.py中line265改成
```
  WPM_SYMBOL_TABLE_FILEPATH = (
      '/tmp/lingvo/lingvo/tasks/asr/wpm_16k_librispeech.vocab')
```
否则在之后会出现`NotFoundError: lingvo/tasks/asr/wpm_16k_librispeech.vocab; No such file or directory`的错误

之后在jupyter notebook的根目录下创建inference.ipynb，输入如下inference代码：
```
import tensorflow as tf
with open('/tmp/librispeech/arctic_a0002.wav') as f:
    read_data = f.read()

from lingvo import model_imports
from lingvo import model_registry
from lingvo.core import inference_graph_exporter
from lingvo.core import predictor
from lingvo.core.ops.hyps_pb2 import Hypothesis
checkpoint = tf.train.latest_checkpoint('/tmp/librispeech/log/train')
print('Using checkpoint %s' % checkpoint)

# Run inference
params = model_registry.GetParams('asr.librispeech.Librispeech960Wpm', 'Test')
inference_graph = inference_graph_exporter.InferenceGraphExporter.Export(params)
pred = predictor.Predictor(inference_graph, checkpoint=checkpoint, device_type='cpu')
hyps, src_frames, en_frames, scores = pred.Run(['hypotheses', 'src_frames', 'encoder_frames', 'scores'], wav=read_data)
print(hyps)
print(src_frames)
print(en_frames)
print(scores)
```
可fetch和feed的值：
Available keys: ['hypotheses', 'src_frames', 'encoder_frames', 'scores']"

Available kwargs:\n             ['wav']"

同时必须要用python自带的read_data读取文件，不能用其他包读取waveform，然后直接feed bytefile，关于inference feed&fetch参见”inference function in /tasks/asr/model.py“

得到结果：
```
Using checkpoint /tmp/librispeech/log/train/ckpt-00019617
[['not at this particular case tom apologise whit more'
  'not at this particular case tom apologized whit more'
  'not at this particular case tom apologised whit more'
  'not at this particular case tom apologise wet more'
  'not at this particular case tom apologies whit more'
  'not at this particular case tom apologised wet more'
  'not at this particular case tom apologised whit moore'
  'not at this particular case tom apologizing']]
[[[[3.8814113]
   [4.70007  ]
   [4.7709765]
   ...
   [7.0900693]
   [7.419367 ]
   [7.3473206]]

  [[4.87256  ]
   [4.780118 ]
   [4.316139 ]
   ...
   [6.676336 ]
   [7.1873636]
   [6.9905477]]

  [[4.522503 ]
   [4.8882885]
   [4.386055 ]
   ...
   [6.823723 ]
   [7.182853 ]
   [7.3419056]]

  ...

  [[4.939454 ]
   [4.6596055]
   [4.743568 ]
   ...
   [7.589517 ]
   [7.509996 ]
   [7.211669 ]]

  [[4.5254383]
   [4.335422 ]
   [4.676789 ]
   ...
   [7.3858976]
   [7.123153 ]
   [7.2415466]]

  [[4.728032 ]
   [3.9538307]
   [3.7164648]
   ...
   [7.249352 ]
   [7.3127604]
   [6.975508 ]]]]
[[[ 1.68075842e-09 -7.73070410e-12 -4.67900099e-06 ... -1.59423719e-18
   -4.81518796e-13  4.24137761e-06]]

 [[ 3.25638544e-11 -8.65923381e-13 -1.76172114e-06 ... -1.20889330e-14
   -0.00000000e+00  5.79266234e-05]]

 [[ 5.16644540e-11 -2.79574280e-12 -5.46445472e-06 ... -1.76007653e-09
   -0.00000000e+00  1.29965262e-03]]

 ...

 [[ 1.01802915e-11 -0.00000000e+00 -8.56795668e-06 ...  0.00000000e+00
    0.00000000e+00  0.00000000e+00]]

 [[ 0.00000000e+00  0.00000000e+00  0.00000000e+00 ...  0.00000000e+00
    0.00000000e+00  0.00000000e+00]]

 [[ 0.00000000e+00  0.00000000e+00  0.00000000e+00 ...  0.00000000e+00
    0.00000000e+00  0.00000000e+00]]]
[[ -1.5081232  -1.5386827  -2.1550722  -3.6003406  -3.8535016  -4.05903
   -4.19409   -10.314721 ]]
```

- [x] standard_vdn
- [x] inference
- [x] train
- [ ] aistation
- [ ] 代码学习
- [ ] 论文学习


## Reference
- http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
- https://docs.bazel.build/versions/master/guide.html
- https://github.com/tensorflow/tensorflow/issues/24496
- https://github.com/tensorflow/lingvo/issues/13
- https://github.com/tensorflow/lingvo/blob/master/codelabs/introduction.ipynb
- https://github.com/tensorflow/lingvo/tree/master/lingvo/tasks/mt
- https://tensorflow.github.io/lingvo/index.html
# Lingvo实验进展
## 环境配置
使用docker进行环境配置，首先需要安装好Docker CE以及nvidia-docker2:
- https://docs.docker.com/install/linux/docker-ce/ubuntu/
- https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)

之后，按照README说明配置docker:
```
LINGVO_DIR="/home/dm/Documents/codes/lingvo"
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
 *** Download Progress Summary as of Wed Apr 24 15:25:23 2019 ***                                      ======================================================================================================
[#7c4ac8 18MiB/5.9GiB(0%) CN:16 DL:304KiB ETA:5h40m20s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-clean-100.tar.gz
------------------------------------------------------------------------------------------------------
[#7f1e27 24MiB/21GiB(0%) CN:16 DL:378KiB ETA:16h30m33s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-clean-360.tar.gz
------------------------------------------------------------------------------------------------------
[#ca61a0 20MiB/28GiB(0%) CN:16 DL:375KiB ETA:22h4m26s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-other-500.tar.gz
------------------------------------------------------------------------------------------------------

 *** Download Progress Summary as of Wed Apr 24 15:26:24 2019 ***                                     
======================================================================================================
[#7c4ac8 39MiB/5.9GiB(0%) CN:16 DL:410KiB ETA:4h11m37s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-clean-100.tar.gz
------------------------------------------------------------------------------------------------------
[#7f1e27 47MiB/21GiB(0%) CN:16 DL:394KiB ETA:15h49m41s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-clean-360.tar.gz
------------------------------------------------------------------------------------------------------
[#ca61a0 41MiB/28GiB(0%) CN:16 DL:321KiB ETA:25h48m23s]
FILE: /home/dm/Documents/datasets/librispeech/raw/train-other-500.tar.gz
------------------------------------------------------------------------------------------------------

......
```
需等待许久，同时下载devtest数据:
```
sh ./lingvo/tasks/asr/tools/librispeech.02.download_devtest.sh
```
实测用迅雷下载更快

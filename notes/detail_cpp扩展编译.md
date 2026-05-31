
```bash
进入虚拟环境（phmr）
cd xxx
   ↓
# 项目路径
export PHMR_PATH=/home/qunz/projects/clone/every-embodied/07-机器人操作、运动控制/Locomotion/video2robot/third_party/PromptHMR
   ↓
安装扩展所需依赖（Eigen）
   ↓
前置准备：
根据项目需求，选定pytorch & cuda版本; 
（注意：RTX 5060 Ti硬件限制：pytorch ≥ 2.7.0 with ≥ cu128(CUDA Runtime>=12.8)
（必要时更新英伟达 Driver）
安装对应版本的pytorch with cu
（前置准备：安装cpp全家桶build-essential）
安装系统级CUDA Tollkit(CUDA Tollkit=CUDA Runtime，至少大版本一致)
   ↓
# 系统级CUDA路径
export CUDA_HOME=/usr/local/cuda-12.8
# (编译,有时运行也需要):nvcc
export PATH=\$CUDA_HOME/bin:\$PATH
   ↓
# 配置编译环境（CPATH）
export CPATH="$CONDA_PREFIX/include/eigen3:${CPATH:-}"
   ↓
前置准备：
编译 sm_120 的 SASS(`setup.py`中的`extra_compile_args`在编译阶段指定GPU架构:`- gencode=arch=compute_低版本,code=sm_120`)
   ↓
编译 C++ 扩展（setup.py）
   ↓
# (运行)优先使用 pytorch 自带 runtime;有些项目会runtime JIT compile CUDA kernel所以第二优先nvcc
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib/python3.10/site-packages/torch/lib:$CUDA_HOME/lib64:$LD_LIBRARY_PATH
# (PYTHONPATH):添加import Python库的搜索路径
export PYTHONPATH=\$PYTHONPATH:\$PHMR_PATH
   ↓
python setup.py install
   ↓
cd xxx
```

---
### 附
#### 1. 前期准备（安装系统级的cuda Toolkit）
```bash
# 进入临时目录下载
cd /tmp

# 下载 CUDA 12.8.0 运行文件 (针对 Linux x86_64)
# 注意：WSL 也可以使用标准的 Linux runfile，关键在于安装时“不选驱动”
wget https://developer.download.nvidia.com/compute/cuda/12.8.0/local_installers/cuda_12.8.0_570.86.10_linux.run

# 安装 CUDA Toolkit
sudo sh cuda_12.8.0_570.86.10_linux.run

# 交互：EULA 协议——accept；Driver——[ ]；CUDA Toolkit 12.8——[X]
```

#### 2. 前期准备（确保`setup.py`可以编译 sm_120 的 SASS）

- 打开`video2robot/third_party/PromptHMR/pipeline/droidcalib/setup.py`
- 修改
```
'nvcc': [
    '-O3',
    '-gencode=arch=compute_120,code=sm_120',
    '-gencode=arch=compute_120,code=compute_120',
    '-gencode=arch=compute_90,code=compute_90',
]
```

#### 3. 前期准备（环境变量配置）
```bash
# conda中环境变量配置(编译cpp扩展)
conda activate phmr

mkdir -p $CONDA_PREFIX/etc/conda/activate.d
mkdir -p $CONDA_PREFIX/etc/conda/deactivate.d

PHMR_PATH="/home/qunz/projects/clone/every-embodied/07-机器人操作、运动控制/Locomotion/video2robot/third_party/PromptHMR"

# activate
cat <<EOF > $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
# 保存旧变量
export OLD_PYTHONPATH=\$PYTHONPATH
export OLD_LD_LIBRARY_PATH=\$LD_LIBRARY_PATH
export OLD_PATH=\$PATH

# 项目路径
export PHMR_PATH=/home/qunz/projects/clone/every-embodied/07-机器人操作、运动控制/Locomotion/video2robot/third_party/PromptHMR
# 系统级CUDA路径
export CUDA_HOME=/usr/local/cuda-12.8

# (编译,有时运行也需要):nvcc
export PATH=\$CUDA_HOME/bin:\$PATH
# (运行)优先使用 pytorch 自带 runtime;有些项目会runtime JIT compile CUDA kernel所以第二优先nvcc
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib/python3.10/site-packages/torch/lib:$CUDA_HOME/lib64:$LD_LIBRARY_PATH

# (PYTHONPATH):添加import Python库的搜索路径
export PYTHONPATH=\$PYTHONPATH:\$PHMR_PATH

EOF

# deactivate
cat <<EOF > $CONDA_PREFIX/etc/conda/deactivate.d/env_vars.sh
export PYTHONPATH=\$OLD_PYTHONPATH
unset OLD_PYTHONPATH

export LD_LIBRARY_PATH=\$OLD_LD_LIBRARY_PATH
unset OLD_LD_LIBRARY_PATH

export PATH=\$OLD_PATH
unset OLD_PATH

unset PHMR_PATH
unset CUDA_HOME
EOF
```


#### Main：编译cpp扩展（droidcalib为例）

```bash
# 编译droidcalib
# source /opt/conda/etc/profile.d/conda.sh
conda install -c conda-forge eigen -y
```

```bash
### ADD:前置准备（此处忽略）

export CPATH="$CONDA_PREFIX/include/eigen3:${CPATH:-}"

### ADD:配置环境变量（上一步已完成）

cd $MY_PHMR_PATH/pipeline/droidcalib
python setup.py install

### ADD:添加LD_LIBRARY_PATH（上一步已完成）
```

#### 备用：重新编译

1. 清理旧编译：
- 进入`phmr`
- `cd video2robot/third_party/PromptHMR/pipeline/droidcalib`
- 执行：
```bash
rm -rf build
rm -rf *.egg-info
```
- 删除旧.so
```bash
rm $CONDA_PREFIX/lib/python3.10/site-packages/droid_backends_intr*.so
```

2. 重新编译：
```bash
### ADD:前置准备（此处忽略）

export CPATH="$CONDA_PREFIX/include/eigen3:${CPATH:-}"

### ADD:配置环境变量（上一步已完成）

cd $MY_PHMR_PATH/pipeline/droidcalib
python setup.py install

### ADD:添加LD_LIBRARY_PATH（上一步已完成）
```

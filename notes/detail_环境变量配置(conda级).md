**参考：**
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

**验证：**
```bash
# 验证配置
conda activate phmr
echo $PHMR_PATH
echo $LD_LIBRARY_PATH
echo $PYTHONPATH
```

---

**系统级vs虚拟环境级 & 为什么要配置在conda里：**
1. `~/.bashrc`是整个Linux系统的配置。即定义了Python 的全局搜索路径（base以及所有虚拟环境）；因此，若`echo 'export PYTHONPATH=$PYTHONPATH:/root/gpufree-data/every-embodied/07-机器人操作、运动控制/Locomotion/video2robot/third_party/PromptHMR' >> ~/.bashrc`，可能下次在另一个项目` import utils`会引用到这个项目的配置！
2. 而在`activate.d/env_vars.sh` 和 `deactivate.d/env_vars.sh` 里配置则在`deactivate`虚拟环境是会将这些配置删除，不影响其他项目。

**配置文件路径：**
1. 系统级环境变量路径：`~/.bashrc`
2. conda-虚拟环境级环境变量路径：`~/miniconda3/envs/phmr/etc/conda/activate.d/env_vars.sh` 以及 `~/miniconda3/envs/phmr/etc/conda/deactivate.d/env_vars.sh`

**代码说明（关键路径更新）：**

1. **`PATH`: 系统nvcc路径，(编译,有时运行也需要)**
	`export PATH=\$CUDA_HOME/bin:\$PATH`
2. **`LD_LIBRARY_PATH`: 编译好的.so运行时查找路径（cpp扩展运行时）** 
	`export LD_LIBRARY_PATH=$CONDA_PREFIX/lib/python3.10/site-packages/torch/lib:$CUDA_HOME/lib64:$LD_LIBRARY_PATH`
	- 优先使用 pytorch 自带 runtime;有些项目会runtime JIT compile CUDA kernel所以第二优先nvcc
	- 注意：系统级 CUDA 库在 `lib64`，Torch 库在 `site-packages/torch/lib`
3. **`PYTHONPATH`: 添加import Python库的搜索路径**
	`export PYTHONPATH=\$PYTHONPATH:\$PHMR_PATH`
	- 配置后：可以直接`import pipeline  import models`
	- 没配置：只能`import PromptHMR.pipeline`

**ps:**
- `\$PYTHONPATH` 注意有`\`! ——避免变量在写文件时被展开
- `$CONDA_PREFIX = ~/miniconda3/envs/phmr`
- `export OLD_???`：保存旧配置
- `export xxx_PATH =a:b:c`
	- `:`——表示路径叠加，先找a路径，再找b，再找c

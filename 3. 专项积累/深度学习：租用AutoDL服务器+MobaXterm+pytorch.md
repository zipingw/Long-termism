## AutoDL服务器+MobaXterm+pytorch



- base 环境下依次执行下列语句 可以看到base环境下安装的torch版本

  ```python
  >>>python
  >>>import torch
  >>>torch.__version__
  ```

  

- 在有卡模式下开机执行`nvidia-smi`可以看到CUDA版本

  

- 服务器已经自带conda，创建虚拟环境并安装pytorch

  ```py
  >>>conda create -n env_name python=3.9
  >>>conda activate env_name
  >>>pip install torch==1.9.1+cu111 torchvision==0.10.1+cu111 torchaudio==0.9.1 -f https://download.pytorch.org/whl/torch_stable.html
  ```
  
- 查看已经创建的虚拟环境

  ```python
  conda env list
  conda info -e
  ```

- 对虚拟环境的操作

  ```python
  # 退出虚拟环境
  >>> deactivate env_name
  >>> source deactivate
  # 切换虚拟环境
  >>> source activate env_name
  # 删除虚拟环境
  >>> conda remove -n env_name --all
  # 删除虚拟环境中的包
  >>> conda remove --name $env_name $package_name
  ```

  










# **复现aloha**

## 配置环境中遇到的问题

主要是以下三个库的安装遇到了问题	

```text
mujoco==2.3.7
dm_control==1.0.14
robomimic
```

对于前两个库 我之前的python版本为3.6 版本太低 不兼容这两个库 将python升级至3.8后成功安装

对于robomimic的安装  一开始对于egl-probe这个包一直报错 直接在pycharm中进行安装也一直失败 后上网查询 **从github上下载源码安装并且安装cmake **之后便成功完成了安装([地址](https://github.com/StanfordVL/egl_probe))  对于通过源码安装的方式 只需解压后进入根目录 在所使用的python环境中打开终端按以下命令行执行即可，(将其中的setup.py进行以下修改)

```
    def build_extension(self, ext):
        extdir = os.path.abspath(os.path.dirname(self.get_ext_fullpath(ext.name)))

        # make build directory
        build_dir = os.path.join(extdir, "egl_probe", "build")
        if os.path.exists(build_dir):
            shutil.rmtree(build_dir)
        os.mkdir(build_dir)

        # build using cmake
        #subprocess.check_call("cmake ..; make -j", cwd=build_dir, shell=True)
        subprocess.check_call("cmake ..", cwd=build_dir, shell=True)
```



```
e.g.
cd egl_probe-master
python setup.py build
python setup.py install
```

但这只是完成了对于egl-probe的安装 想安装robomimic还需要离线安装**diffusion-policy-mg** [点这里](https://github.com/ARISE-Initiative/robomimic/tree/diffusion-policy-mg)

最后再安装detr所需要的util库，这个库已经在源码的 **act-plus-plus-main\detr\util**路径下 复制整个文件夹将其放入所使用的python环境路径下即可完成整个环境配置

```
#示例 我的python虚拟环境路径
D:\pythonProject\final\Scripts
```

## 数据集的构建	

数据集的采集有两种方式 一种是生成仿真数据 一种是采用项目自带的数据

在这之前 先在以下两处地方修改代码

```
e.g. constants.py 中修改数据路径
DATA_DIR = 'D:/aloha/data1'
#detr_vae 第291行处进行以下修改
 # encoder = build_transformer(args)
        encoder = build_encoder(args)
```

1. **采用仿真数据**

	在终端中输入以下命令即可

	```
	python record_sim_episodes.py --task_name sim_transfer_cube_scripted --dataset_dir data/sim_transfer_cube_scripted --num_episodes 10  --onscreen_render
	
	#之后保存数据
	python visualize_episodes.py --dataset_dir data/sim_transfer_cube_scripted --episode_idx 0
	```

2. **采用项目自带的数据**

	进入网址[下载](https://drive.google.com/drive/folders/1gPR03v05S1xiInoVJn7G7VJ9pDCnxq9O)项目自带的数据集并解压到你所指定的目录，但若直接进行训练会出现以下报错

	```
	Error loading /home/lenovo/act-plus-plus-main/data/sim_transfer_cube_scripted/episode_xx.hdf5 in __getitem__
	```

	解决方案：进入constant.py进行以下修改

	```
	    'sim_transfer_cube_scripted':{
	        'dataset_dir': DATA_DIR + '/sim_transfer_cube_scripted',
	        'num_episodes': 50,
	        'episode_len': 400,
	        #'camera_names': ['top', 'left_wrist', 'right_wrist']
	        'camera_names': ['top']
	    },
	```

	官方的数据只有top视角 所以需要将后面两个视角给注释掉

## 训练

​	训练之前，我们首先先注册一个wandb帐户 也便于日后学习使用

​	注册登录之后 获取自己的API key 在python终端下

```
pip install wandb
wandb login
#粘贴你的API key进入即可
```

  	在imitate_episode.py中的155行修改代码

```
#wandb.init(project="alohaa", reinit=True, entity="moma", name=expr_name)
#将project改成你自己创建的名字
wandb.init(project="alohaa", reinit=True,name=expr_name)
```

​	接下来我们就可以开始训练啦

```
#运行以下代码进行训练
python imitate_episodes.py --task_name sim_transfer_cube_scripted --ckpt_dir trainings --policy_class ACT --kl_weight 1 --chunk_size 10 --hidden_dim 512 --batch_size 1 --dim_feedforward 3200  --lr 1e-5 --seed 0 --num_steps 2000
```


## 前期准备

1. 安装CUDA,cudnn
   - cuda版本与驱动匹配（桌面右键->NVIDIA控制面板->帮助->系统信息->组件 查看）
   - cudnn版本与cuda匹配
2. 创建conda环境，安装pytorch

> 注意：python 版本需要`3.7`及以上
>
> 上述步骤参考博客：[1050ti显卡安装cuda_VSCODE2021的博客-CSDN博客_1050ti cuda](https://blog.csdn.net/weixin_44739865/article/details/121460260)

3. 在3D Slicer中安装Monailabel插件

注：所有文件夹名不能有空格，可用下划线代替，不要有中文字符（可能没法识别）

------------



## 安装monailabel

1. 激活conda环境，安装monailabel包

```bash
pip install monailabel 
```

2. 下载功能包

下载radiology功能包，其中包含了segmentation, deepedit, deepgrow三个主要的工具

> 执行下列命令前，先将anaconda prompt定位到工作目录（自己确定）下，功能包会下载到当前工作目录下，此目录需要牢记
>
> ![image-20221122214426848](monailabel.assets/image-20221122214426848.png)

```bash
monailabel apps --name radiology --download --output .
```

3. 启动服务

```bash
monailabel start_server --app $directory_of_radiology$ --studies $directory_of_data_file$ --conf models segmetation
```

看到下列信息表示服务启动成功

![image-20221122214138735](monailabel.assets/image-20221122214138735.png)

> --app 后的参数是radiology文件夹的目录
>
> --studies 后是需要处理的数据所在文件夹

更多可选参数详见[MONAILabel/sample-apps/radiology at main · Project-MONAI/MONAILabel · GitHub](https://github.com/Project-MONAI/MONAILabel/tree/main/sample-apps/radiology)

4. 打开Slicer中的monailabel插件，连接上一步启动的monailabel服务

### Slicer中的设定

- 选择**Edit->Application settings**

![image-20221122214709905](monailabel.assets/image-20221122214709905.png)



- 选择**MONAI Label**, 勾选**Developer Mode**

![image-20221122214903229](monailabel.assets/image-20221122214903229.png)

- 选择**Modules**，右边找到**MONAILabel**，将其拖到下方**Favorite Modules**，然后重启Slicer

![image-20221122215122683](monailabel.assets/image-20221122215122683.png)

再次打开Slicer就可以看到MONAILabel的图标在快速工具栏

![image-20221122215239124](monailabel.assets/image-20221122215239124.png)

---------



## 训练模型

### 修改配置文件

在radiology文件夹下，打开lib->configs->segmentation.py，修改labels，并将使用与训练模型这个参数改成**false** 

![image-20221122221715298](monailabel.assets/image-20221122221715298.png)

configs, trainers, infers三个文件夹下的segmentation.py都有一个target spacing 参数，原始值是（1，1，1），将其改成自己图片的spacing，如（0.4，0.4，0.4）

![image-20221122221945379](monailabel.assets/image-20221122221945379.png)

### 准备数据

1. 已经标注了一批数据，在原图像文件夹下新建一个**labels**文件夹

![image-20221122215708582](monailabel.assets/image-20221122215708582.png)

​	然后在labels文件夹下再新建一个**final**文件夹，将标注好的label放到该文件夹下

![image-20221122215811561](monailabel.assets/image-20221122215811561.png)

> original文件夹可以不建，label的文件名需要与image完全一样

**回到slicer**

![image-20221122220217581](monailabel.assets/image-20221122220217581.png)

该进度条表示已标注数据的比例

2. 从0开始标注数据

   点击**Next Sample**抓取数据，并点击**Segment Editor**进行标注

   ![image-20221122222015898](monailabel.assets/image-20221122222015898.png)

![image-20221122222610790](monailabel.assets/image-20221122222610790.png)

标注完成后点击**Submit Label**，然后抓取下一张未标注图片

### 开始训练

在**Options** 下，将**Section**选择为**train**，然后更改**max_epochs**，其他参数一般不需要改动

![image-20221122222311448](monailabel.assets/image-20221122222311448.png)

然后就可以点击**Train**开始训练了

### 完成训练

完成训练后，再次抓取未标注图片，然后在**Auto Segmentation**下点击**Run**就可以开始预测了


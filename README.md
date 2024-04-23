## 老视频修复简介

老照片/视频记录着曾经的岁月，承载着美好的回忆与厚重的历史。但由于年代久远，旧的影像素材往往存在模糊、缺色、跳帧等问题。

我们针对于上述提到的问题运用PaddleGAN中现有的技术进行上色、超分及补帧。

## PaddleGAN中要使用的预测模型介绍
### 补帧模型DAIN
DAIN 模型通过探索深度的信息来显式检测遮挡。并且开发了一个深度感知的流投影层来合成中间流。在视频补帧方面有较好的效果。
![](./imgs/dain_network.png)

```
ppgan.apps.DAINPredictor(
                        output_path='output',
                        weight_path=None,
                        time_step=None,
                        use_gpu=True,
                        remove_duplicates=False)
```
#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`。
- `time_step (int)`: 补帧的时间系数，如果设置为0.5，则原先为每秒30帧的视频，补帧后变为每秒60帧。
- `remove_duplicates (bool，可选的)`: 是否删除重复帧，默认值：`False`.

### 上色模型DeOldifyPredictor
DeOldify 采用自注意力机制的生成对抗网络，生成器是一个U-NET结构的网络。在图像的上色方面有着较好的效果。
![](./imgs/deoldify_network.png)

```
ppgan.apps.DeOldifyPredictor(output='output', weight_path=None, render_factor=32)
```
#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`。
- `render_factor (int)`: 会将该参数乘以16后作为输入帧的resize的值，如果该值设置为32，
                         则输入帧会resize到(32 * 16, 32 * 16)的尺寸再输入到网络中。

### 上色模型DeepRemasterPredictor
DeepRemaster 模型基于时空卷积神经网络和自注意力机制。并且能够根据输入的任意数量的参考帧对图片进行上色。
![](./imgs/remaster_network.png)

```
ppgan.apps.DeepRemasterPredictor(
                                output='output',
                                weight_path=None,
                                colorization=False,
                                reference_dir=None,
                                mindim=360):
```
#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`。
- `colorization (bool)`: 是否对输入视频上色，如果选项设置为 `True` ，则参考帧的文件夹路径也必须要设置。默认值：`False`。
- `reference_dir (bool)`: 参考帧的文件夹路径。默认值：`None`。
- `mindim (bool)`: 输入帧重新resize后的短边的大小。默认值：360。

### 图像超分辨率模型RealSRPredictor
RealSR模型通过估计各种模糊内核以及实际噪声分布，为现实世界的图像设计一种新颖的真实图片降采样框架。基于该降采样框架，可以获取与真实世界图像共享同一域的低分辨率图像。并且提出了一个旨在提高感知度的真实世界超分辨率模型。对合成噪声数据和真实世界图像进行的大量实验表明，该模型能够有效降低了噪声并提高了视觉质量。

![](./imgs/realsr_network.png)

```
ppgan.apps.RealSRPredictor(output='output', weight_path=None)
```
#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`。

### Real-SR是腾讯优图实验室提出一种新的图像超分辨率算法。该算法在CVPR-NTIRE-2020真实图像超分比赛中以明显优势获得双赛道冠军。

与已有的超分辨率方法相比，RealSR的创新主要体现在三个方面：

1. RealSR采用了自主设计的新型图片退化方法，通过分析真实图片中的模糊和噪声，模拟真实图片的退化过程

2. 不需要成对的训练数据，利用无标记的数据即可进行训练。

3. 可以处理低分辨率图像中的模糊噪声问题，得到更加清晰干净的高分辨结果。

算法的主要步骤可以分为两个模块：退化模型的估计，超分模型的训练。方法框架如下图所示
————————————————

                            版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
                        
原文链接：https://blog.csdn.net/LU__JH/article/details/117432763![image](https://github.com/aimerfeng/PaddleGAN-BasicVSR--/assets/59301770/1bf02f86-5cbb-44f9-887c-6594962500db)

#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`.
- `num_frames (int，可选的)`: 模型输入帧数，默认值：`10`.输入帧数越大，模型超分效果越好。

### 视频超分辨率模型BasicVSR系列
BasicVSR在VSR的指导下重新考虑了四个基本模块（即传播、对齐、聚合和上采样）的一些最重要的组件。 通过添加一些小设计，重用一些现有组件，得到了简洁的 BasicVSR。与许多最先进的算法相比，BasicVSR在速度和恢复质量方面实现了有吸引力的改进。 

同时，通过添加信息重新填充机制和耦合传播方案以促进信息聚合，BasicVSR 可以扩展为IconVSR，IconVSR可以作为未来 VSR 方法的强大基线 .

BasicVSR++通过提出二阶网格传播和导流可变形对齐来重新设计BasicVSR。通过增强传播和对齐来增强循环框架，BasicVSR++可以更有效地利用未对齐视频帧的时空信息。 在类似的计算约束下，新组件可提高性能。特别是，BasicVSR++ 以相似的参数数量在 PSNR 方面比 BasicVSR 高0.82dB。BasicVSR++ 在NTIRE2021的视频超分辨率和压缩视频增强挑战赛中获得三名冠军和一名亚军。

![](./imgs/basicvsr++_network.jpg)

```
ppgan.apps.BasicVSRPredictor(output='output', weight_path=None, num_frames)
```
```
ppgan.apps.IconVSRPredictor(output='output', weight_path=None, num_frames)
```
```
ppgan.apps.BasiVSRPlusPlusPredictor(output='output', weight_path=None, num_frames)
```
#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`.
- `num_frames (int，可选的)`: 模型输入帧数，默认值：`10`.输入帧数越大，模型超分效果越好。

### 超分辨率模型EDVR
EDVR模型提出了一个新颖的视频具有增强可变形卷积的还原框架：第一，为了处理大动作而设计的一个金字塔，级联和可变形（PCD）对齐模块，使用可变形卷积以从粗到精的方式在特征级别完成对齐；第二，提出时空注意力机制（TSA）融合模块，在时间和空间上都融合了注意机制，用以增强复原的功能。
![](./imgs/edvr_network.png)

```
ppgan.apps.EDVRPredictor(output='output', weight_path=None)
```
#### 参数

- `output_path (str，可选的)`: 输出的文件夹路径，默认值：`output`.
- `weight_path (None，可选的)`: 载入的权重路径，如果没有设置，则从云端下载默认的权重到本地。默认值：`None`。

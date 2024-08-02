文章给出了数据集发展的四个重点：large-scale(no mocap datas are needed)、whole-body motion、text annotations、 multi-scenarios，而且文章处理的是视频序列

***全身动作标注：***

**2D Keypoint Estimation阶段ViT-based model**：先使用ViT-WholeBody粗略获取全身各keypoints的位置和置信度，然后利用手部和面部的keypoints的信息求出手和脸的bounding box（用BodyHands detector来优化）实现手、面、身的裁剪，分别输入三个预训练的ViT net以获取不同部位的keypoints然后以此更新whole-body keypoints。

**Score-guided Adaptive Smoothing**：使用基于置信度的滑窗smooth，对置信度较低的关键点使用较大的窗口尺寸。

**3D Keypoint Annotation**：对于单目视频用预训练模型（Learning 3D Human Pose Estimation from Dozens of Datasets using a Geometry-Aware Autoencoder to Bridge Between Skeleton Formats）对于多目视频，先使用BA获取相机参数，使用三角形法利用多目的2d关键点重建3D关键点。提高稳定性：时间平滑、重建时加强3d骨骼的长度限制（每块骨头的长度在整个视频中不能变）

**Local Pose Optimization.**：**有一个关键的地方（本文keypoints有133个而smplx加上root只有54个kp）核心的idea就是利用这检测出的更多的keypoints对pose的预测做优化**。这里就是先利用SOTA models（OSX：全身、EMOCA：面部）求出smplx的参数（作为pose先验）【个人不是很接受这个参数的设置】，然后初始化一个Θ*，利用这个参数，重建smplx，再利用一个线性回归器（我没看到具体是什么样）重建出的模型的133keypoints与原来识别的keypoints比较，将133投影到2D再进行一次比较，自此产生loss joints的三个部分（3D 2D prior）。整体的loss除了loss joints还有smooth、pen（collision：出自smplx）、phy（implausible pose：出自PhysCap）

**Global Motion Optimization**.（based on GLAMR）：2D点误差、全局轨迹和Kama预测的轨迹的误差、相机平滑误差、全局轨迹正则误差

***获取全身动作描述***

动作序列label：用action数据集本身有的label，喂给Video-LLaMA再将人类行为描述过滤为补充文本，用easyocr自动提取字幕。用LLM产生的search query作为语义标记。没有语义信息的视频就用VGG Image Annotator (VIA)手动标注

全身姿势描述：用EMOCA分类识别面部信息。主体的描述利用了postscript。通过手指弯曲度和手指之间的距离定义了六种基本手指姿势，以生成描述。据三维手部关键点计算每个手指关节的角度，并确定相应的边距。

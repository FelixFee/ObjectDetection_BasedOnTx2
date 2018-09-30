# Object Detection Research Based on TX2
Lately our lab get a new device:TX2.My job is to do some research around implementing Object Detecton Algorithms on it. 

TX2, whose whole name is Nvidia Jetson TX2, is a powerful embedded system(SoC).It has 6 cpu cores and a gpu with 256 cuda cores.More details can be found [here](https://www.nvidia.com/zh-cn/autonomous-machines/embedded-systems-dev-kits-modules/).
Its computing power may not be able to beat GTX 1070ti but is enough for some actual applications.

Object Detection is a category of Computer Vision which aims at recognizing objects in images and simultaneously reporting object`s position and bounding box.

Here is an example.
![image](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=1233252412,2871820223&fm=26&gp=0.jpg)
And another example.
![image](https://pjreddie.com/media/image/Screen_Shot_2018-03-24_at_10.53.04_PM.png)

It is easy to understand what the term 'detection' means after seeing an example.It means the combination of **Classification** and **Localization**.

![image](http://mmbiz.qpic.cn/mmbiz_png/iaTa8ut6HiawDhWYblXp7Uqo1KKTNzCzzRITWA48CsUGcnVUiayPmfGW00KF7ia6nXPguAYLVpicTYZ3EMOusgT5Y5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Current Object Detection technique can be subdivided into two  classes: one-stage methods and two-stage methods.The two-stage methods divide the problem into two steps:1 .Find appropriate area. 2. Use classifiers to recognize what\`s in the area.Howeverr, one-stage methods simply use a unified CNN network to predict both objects\` class and location.Usually speaking, one-stage methods run faster and their models are simpler.Classic one-stage model includes YOLO(you only look once),SSD(single shot detection),retina-net and so on.These models and their variation are considered to be good for embeded system like TX2.

After some researches, I found yolov3、RFBNet、 a very compromising model, because it seems to be very fast and both accurate.

Download their papers here.<br>
YOLO - https://arxiv.org/abs/1506.02640<br>
YOLO9000 - https://arxiv.org/abs/1612.08242<br>
YOLOv3 - https://arxiv.org/abs/1804.02767<br>
SSD - https://arxiv.org/abs/1512.02325<br>
RFBNet - https://arxiv.org/abs/1711.07767<br>
RetinaNet - https://arxiv.org/pdf/1708.02002.pdf<br>

=

![image](https://pjreddie.com/media/image/map50blue.png)
*This result come from pjreddie.com*

Then you can clone the newest yolov3 repo、modify Makefile and make.Details can be found in this [blog](https://jkjung-avt.github.io/yolov3/)
Assuming everything went OK, you should be able to  run test on your TX2 board!
But I found that yolov3 is really slow on TX2.The first time I run it, I only get 2\~3 fps(or 0.3\~0.5 sec).I have run nvpmodel and jetson_clocks already.So we decided to check something else.
The best thing I found is that you can set the model\`s input size in yolov3.cfg file.The input size can vary from 144 to 608.(The size must be 32 x ratio).I didn\`t test bigger or smaller size.As to our research I guess that\`s enought.Too small make model useless while too big is not bearably slow.
So let\`s try to decrease the default size(416) and see whether we can find a good trade-off.
I do this test by using yolov3 model(.cfg) and voc data.Sadly I found that there is no yolov3-voc weights available, the official yolov3.weights is trained on coco data.So I trained my own yolov3-voc weights(10400 iterations , used yolov3.weights as pretrained model).
And use this weights to calculate mAP.
Here is the result I have(Size vary from 160-384).

![image](https://github.com/FelixCaae/ObjectDetection_BasedOnTx2/blob/master/fps_mAP.png)

So the result is interesting, the fourth point and fifth point has nearly same perfomance but totally different speed.There is a gap between two group of points.And so I think the fifth point(size 256) is a good choice to speed up yolov3`s perfomance.


```python
import cv2 #opencv导入 ，一般opencv都是BGR格式 [高度，宽度，通道 (3)]
import numpy as np
from hobot_vio import libsrcampy # RDK官方的包

PIPE_ID = 0
# IMX219原生分辨率
WIDTH = 1920
HEIGHT = 1080
# NV12格式高度是原图的1.5倍
YUV_TOTAL_HEIGHT = int(HEIGHT * 1.5)

cam = libsrcampy.Camera() # 创建camera类
# open_cam填写原生支持的1920×1080
# 需要注意，我们用python写opencv，MIPI摄像头只能读取原生分辨率，这里统一设置1920和1080，后续再用opencv缩放尺寸
ret = cam.open_cam(pipe_id=PIPE_ID, video_index=-1, fps=30, width=WIDTH, height=HEIGHT)
if ret != 0:
    print("摄像头打开失败！")
    exit()

print("摄像头开启成功，开始读取画面")
while True:
    # 获取一帧NV12图像，超时没有获取到图像返回None，拿到画面立刻返回
    img_bytes = cam.get_img(2)
    if img_bytes is None:
        continue  # 容错机制，防止后续代码报错，继续读取下一帧图像
    
    # 将字节数据转为np数组，再转换为OpenCV识别的BGR格式
    yuv_np = np.frombuffer(img_bytes, np.uint8).reshape((YUV_TOTAL_HEIGHT, WIDTH))
    frame_bgr = cv2.cvtColor(yuv_np, cv2.COLOR_YUV2BGR_NV12)
    # 软件缩放到640×380
    frame_bgr = cv2.resize(frame_bgr, (640, 380))
    #----算法插入部分------
    
    #--------------------
    cv2.imshow("Camera", frame_bgr) # 相当于打开一个title为Camera的窗口展示frame_bgr
    key = cv2.waitKey(1) & 0xFF
    if key == ord('q'):
        break

cam.close_cam()
cv2.destroyAllWindows()
```

# 1. 灰度图等的转化
`gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)`
这个是转为灰度图，把frame转为一个灰度图
- `cv2.COLOR_BGR2HSV`   HSV
- `cv2.COLOR_BGR2LAB`   LAB
## 二值化算法
1. `mask = cv2.inRange(hsv, lower, upper)`
   对HSV图像做二值化，lower和upper是两个阈值，分别是两个列表\[H,S,V]
2. `ret, binary = cv2.threshold(src, thresh, maxval, type)`
   这个是灰度二值化，就是灰度图；ret返回实际使用的阈值，binary返回输出的二值图像
   - `src`：**只能输入单通道灰度图，不能传 BGR 三通道图片**；
   - `thresh`：手动设定阈值（0‑255）；OTSU 自动二值化时写 0；
   - `maxval`：超过阈值后赋予的值，二值化固定填`255`；
   - `type`：阈值模式
     `cv2.THRESH_BINARY`  像素＞阈值 →255；否则 0（最常用）
     `cv2.THRESH_BINARY_INV`  像素＞阈值 →0；否则 255（取反）

3. OTSU 自动阈值
   
4. 自适应阈值
   `binary = cv2.adaptiveThreshold(src, maxVal, adaptiveMethod, thresholdType, blockSize, C)` 
   
# 2. 图像处理

## 1.滤波
滤波适用于彩色图和灰度图，能有效去除图形中的白色噪点等(作用于彩色图(BGR图)，减少原图噪点，提高颜色提取质量)
比如BRG转HSV去识别颜色，我们在BRG转HSV的代码中间加上一个滤波，就会让转化质量大大提高；边缘检测需要在转为灰度图后再用高斯滤波
我们可以认为，滤波是对 RGB这三个通道分别进行的
### 1. 均值滤波
`img = cv2.blur(img, (3,3))`
第一个参数是对哪个图像操作，第二个参数是卷积核的大小

原理就是通过卷积核滑动扫过整个图像，对每一个卷积核内的BRG三个通道分别单独累加平均，赋值给中心
这样会把一些白色的噪点和周围的颜色进行平均，但是坏处是形态边缘被模糊，此时转为灰度图的效果会变差
优势：算法极简，算力消耗最低，能柔和抹平细碎噪点。
缺陷：不区分边缘与背景，整张图均匀模糊，物体轮廓、色块边缘会严重虚化，后续灰度图、轮廓检测效果变差。
### 2.高斯滤波
`img = cv2.GaussianBlur(src, ksize, sigmaX, sigmaY=None)`
第一个参数是图像，第二个参数是卷积核 ，第三个参数默认0就好 ，第四个参数可以不写

`sigmaX=0.8`：控制模糊的强弱，数值越大，模糊越重、降噪越强。
数值越大，算力吃的越狠

也叫正态滤波，相当于对均值滤波做一个权重计算，越靠近整个数的关联越大，比较吃算力
画面是细密均匀颗粒噪点：**高斯滤波表现更好，中值反而会破坏物体形状**
而且会模糊边缘信息
适用场景：画面布满均匀细密颗粒噪点，高斯平滑效果最优；
缺陷：依旧会模糊物体边缘，只是比均值滤波轻微；算力高于均值滤波。

### 3.中值滤波
`img = cv2.medianBlur(src, ksize)`
参数同理

把一个卷积核中中间的值当作结果，能把一个有白色噪点的图彻底变无噪点，画面白点、黑点噪点多时：**中值滤波效果优于高斯滤波**；
优势：对孤立黑白椒盐噪点（镜头灰尘、白点黑点）去除能力极强，效果远优于高斯 / 均值；

缺陷：会腐蚀细小轮廓，圆形、细线类目标容易变形，边缘保留能力差。
### 4.双边滤波
```
dst = cv2.bilateralFilter(src, d, sigmaColor, sigmaSpace)
```

1. `src`：输入图像，只能是 8bit 彩色图 (BGR) 或灰度图；
2. `d`：滤波窗口直径，填 5/9 即可，填 0 会自动根据 sigmaSpace 计算；
3. `sigmaColor` 色彩权重标准差：
    像素之间颜色差距越小，权重越高；数值越大，允许色差更大的像素一起平滑；
4. `sigmaSpace` 空间权重标准差：
    离窗口中心越近的像素权重越高；数值越大，更远像素参与模糊。

优点是降噪的同时保留边缘细节，做轮廓识别等需要边缘的非常好
缺点是最吃算力的一个
优点：**同时结合空间距离 + 色彩差异双重权重**，只平滑同色背景噪点，色块、目标轮廓边缘完整保留；做 HSV 颜色分割、霍夫圆、轮廓识别场景效果最好。

缺点：计算逻辑复杂，是四种滤波里**算力开销最大**的，高分辨率全图使用容易拉高 CPU 占用、造成卡顿。
****
## 2. 图像形态学操作
这种操作只适用于灰度图或者二值图 mask
### 1.腐蚀操作
```python
kernel = np.ones((5,5),np.uint8)
erosion = cv2.erode(src,kernel,iterations) # iterations是循环次数，原图像会保留
```
在一个图像中，有一个卷积核，在卷积核滑动过程中，如果里面全是255（白色），那中心就是白色，但凡有一个黑，那中心就是黑色(卷积核越大，图像越小，腐蚀越厉害)
（OpenCV 做腐蚀时，会把原图复制一份作为参考模板；kernel 滑动遍历每一个位置时，取值全部取自原始 mask，临时修改的像素不会参与后续判断。）

### 2. 膨胀操作
```python
kernel = np.ones((5,5),np.uint8)
dilate = cv2.dilate(src, kernel, iterations=1)
```
在一幅图像里面设置一个卷积核，卷积核在图片上滑动遍历；

只要卷积核范围内**任意一个像素是 255（白色），中心点就判定为白色；只有卷积核里面全部像素都是黑色 (0)，中心点才会是黑色**。卷积核尺寸越大，白色区域向外扩张越明显，物体膨胀效果越强。

Open‑CV 执行膨胀时，会复制一份原图当作参考模板；kernel 滑动遍历每一处位置，取值全部来自原始 mask，本次运算里刚修改后的像素不会用来判断旁边像素；全部位置计算完毕之后，统一输出膨胀后的图片。

### 3.开运算和闭运算
开运算就是==**先腐蚀后膨胀**==
```python
	kernel = np.ones((5,5),np.uint8)
	opening = cv2.morphologyEx(src, cv2.MORPH_OPEN,kernel，iterations=1)
```
闭运算就是先膨胀后腐蚀
```python
kernel = np.ones((5,5),np.uint8)
opening = cv2.morphologyEx(src, cv2.MORPH_CLOSE,kernel，iterations=1)
```

- **开运算：腐蚀→膨胀（MORPH_OPEN）**
    
    - 作用：清除背景里面零散细小的白色噪点；物体整体大小基本不变；
    - 适用：背景到处是小白斑，小球本体完好（就是你现在代码用的）。
    
- **闭运算：膨胀→腐蚀（MORPH_CLOSE）**
    
    - 作用：填补物体内部小黑洞、细小裂痕；
    - 适用：目标物体内部有黑色空缺。

如果两者都有，就先开运算，再闭运算

****

## 3. 图像梯度计算

### 1. Sobel 算子
```python
dst = cv2.Sobel(src, ddepth, dx, dy, ksize)
```
本质上Sobel算子是一个已经固定好的卷积核，在滑动的时候会对应位置相乘后相加，也叫做卷积运算（不是矩阵乘法），ddepth就是代表计算结果的数据类型，dx和dy取1/0，表示计算x还是y的梯度，不建议同时取1，推荐分别算然后用 2.梯度计算方法 的方法去求和，效果会好。ksize = 3表示卷积核的大小

输出的dst是一个图片每个像素都满足这个算法
- 平坦区域：`sobelx[y][x]≈0`；
- 黑‑白边缘：数值 > 0；
- 白‑黑边缘：数值 < 0；
但是opencv在imshow时会把小于0的地方截断为0，导致无法表示差距关系
```python
sobelx = cv2.Sobel(img,cv2.CV_64F,1,0,3)
sobelx = cv2.convertScaleAbs(sobelx)
```
所以一般这么写，ddepth=cv2.CV_64F，表示是浮点数，由正负，然后下一步`sobelx = cv2.convertScaleAbs(sobelx)` 是取绝对值并将`CV_64F`浮点型转回`uint8(0‑255)`格式，适配图片显示，这样就保留了完整的梯度关系

### 2. 梯度计算方法

我们分别计算了x和y的方向梯度关系，一般我们要求梯度会选择
$G = \sqrt{(G_x)^2+(G_y)^2}$
这是一种比较精确的算法，占用算力也较大，opencv的公式为
```python
gx = cv2.Sobel(gray, cv2.CV_64F, 1, 0, 3)
gy = cv2.Sobel(gray, cv2.CV_64F, 0, 1, 3)

g = cv2.magnitude(gx, gy) # 这一步等价于上面的公式
g = cv2.normalize(g, None, 0, 255, cv2.NORM_MINMAX, cv2.CV_8U)
```
`g = cv2.normalize(g, None, 0, 255, cv2.NORM_MINMAX, cv2.CV_8U)`
1. **src：g（输入数组）**
    
    就是经过`magnitude`得到的梯度浮点矩阵（`float64`）。里面数值全部≥0，但是数值范围不确定：最小值可能是 0，最大值可能几百甚至上千，不在 0‑255 范围内，没法直接当作图片显示
1. **dst = None**
    输出结果存放位置；写`None`代表函数会新建数组，结果赋值给左侧变量`g`；你也可以预先创建矩阵传进去。
2. **alpha = 0**
	在`NORM_MINMAX`模式下：数据缩放后的**最小值**。
    最后矩阵里原来的最小值 → 映射成 alpha（0）。
3. **beta = 255**
    `NORM_MINMAX`模式下：缩放之后的**最大值**。
    原来矩阵中的最大值 → 映射成 beta（255）。
4. **norm_type = cv2.NORM_MINMAX（归一化模式，重点）**
    把所有的值都映射到0-255的范围
5. **`cv2.CV_8U`**
	指定输出的数据类型，`CV_8U`也就是`uint8`，取值范围 0‑255，转换为 OpenCV 标准图片格式。

此时就不用去绝对值了，因为第一个求和也去掉负数关系

还有一种算法就是x和y的梯度加绝对值求和
```python
sobelx = cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)
sobelx = cv2.convertScaleAbs(sobelx)

sobely = cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)
sobely = cv2.convertScaleAbs(sobely)

sobelxy = cv2.addWeighted(sobelx, 0.5, sobely, 0.5, 0)
cv_show(sobelxy, 'sobelxy')
```
这个就要先取绝对值在求和，0.5是权重，那个0是偏置，一般不加
### 3. 其余算子

****
## 4. 边缘检测流程（Canny）

### 1. 用法
```python
```

## 5. 轮廓检测

### 1.轮廓检测方法和结果
通常寻找轮廓就用一个函数 `contours, hierarchy = cv2.findContours(img, mode, method)`
为保证检测结果，img必须采用二值图，寻找白色区域的轮廓
- **contours：列表**
    里面每一个元素对应一个轮廓；每个轮廓是形状为`[N,1,2]`的 numpy 数组，`N`是轮廓点的数量，存储`(x,y)`坐标。
- **hierarchy：层级信息，形状为`(1,总轮廓数,4)`**
    4 个含义：`[next, previous, child, parent]`（下一个轮廓、前一个轮廓、子轮廓、父轮廓）。
    
mode:轮廓检索模式
• RETR_EXTERNAL：只检索最外面的轮廓;
• RETR_LIST：检索所有的轮廓，并将其保存到一条链表当中;
• RETR_CCOMP：检索所有的轮廓，并将他们组织为两层：顶层是各部分的外部边界，第二层是空洞的边界;
• RETR_TREE：检索所有的轮廓，并重构建嵌套轮廓的整个层次;  
==\[平常用第一个和第四个就够了]==

method:轮廓逼近方法
• CHAIN_APPROX_NONE：以Freeman链码的方式输出轮廓，所有其他方法输出多边形（顶点的序列）。 其实就是把罗坤上的所有点都存起来
• CHAIN_APPROX_SIMPLE:压缩水平的、垂直的和斜的部分，也就是，函数只保留他们的终点部分
****

然后就是绘制函数`res=cv2.drawContours(image, contours, contourIdx, color, thickness)`
第一个参数表示在哪个图片上画
第二个参数就是轮廓点的列表
第三个参数就是画第几个轮廓，-1是全部
第四个参数就是颜色，BRG格式，(B,G,R)
第五个参数是线条宽度，-1是填充

注意，每一次绘制都会改变image的图像，即image的原图就没了(res和image 指向同一个图片，res也可以不写)，如果需要原图，可以`image_copy = image.copy()`

#### 2. 画标准图形
1.  水平矩形
   `cv2.rectangle(img, pt1, pt2, color, thickness, lineType=None, shift=None)`
   pt1，pt2分别代表左上角和右下角的坐标。


### 2. 轮廓特征和近似
`con`代表`findContours`得到的单个轮廓。
#### 1. 轮廓面积 `cv2.contourArea(con)`

```python
area = cv2.contourArea(con)
```
1. 作用：计算轮廓包围区域的像素面积；
2. 工程用途：过滤噪声轮廓，面积过小直接舍弃。

```python
if area > 80:
    #才认为物体有效
```
注意：只计算闭合轮廓内部像素。
****
#### 2. 轮廓周长 `arcLength()`
```python
#第二个参数 True代表轮廓闭合图形
peri = cv2.arcLength(con, True)
```

- 返回轮廓一圈的长度；
- 后续多边形逼近要用到周长。
****
#### 3. 多边形逼近 `approxPolyDP（重点）`
```python
approx = cv2.approxPolyDP(curve, epsilon, closed)
```

1. 参数：
    - `curve`：原始轮廓 con；
    - `epsilon`：逼近精度，**一般取周长的百分比，常用 `0.04 * peri`**；epsilon 值越大顶点越少；
    - `closed=True`：轮廓闭合。
2. 返回值 `approx`：保存多边形顶点，格式`(N,1,2)`，`N`就是顶点个数，依靠顶点数量判断图形：（len(approx)）
    - N = 3：三角形
    - N = 4：矩形、正方形
    - N＞6：判定圆形。

```python
peri = cv2.arcLength(con,True)
epsilon = 0.04*peri
approx = cv2.approxPolyDP(con,epsilon,True)
corner_num = len(approx)
```

****
#### 4. 外接矩形 `boundingRect`
```python
x, y, w, h = cv2.boundingRect(con)
```
- `(x,y)`：矩形左上角坐标；
- `w`宽度，`h`高度；
    之后用`cv2.rectangle()`画出方框。

还有一种是最小邻接矩形，可以旋转，具体可以去看[[2. Opencv 对图像处理操作#2. 绘制]]里面的代码。
****
#### 5.图像矩 moments（求物体中心点）
```python
M = cv2.moments(con)
#m00是面积
if M["m00"] != 0:
    cx = int(M["m10"] / M["m00"])
    cy = int(M["m01"] / M["m00"])
```

M是一个字典
```python
{ "m00": 数值, "m10": 数值, "m01": 数值, "m20":..., "m02":..., "m11":..., …… }
```
m00 : 对应的是里面所有的像素的总数，等价于轮廓面积。 （和`cv2.contourArea(con)`有细微差别）
m10：一阶矩，全部像素 x 坐标累加和
m01：一阶矩，全部像素 y 坐标累加和

再用质心公式，即可求得中心，具体公式看代码。@ 二阶矩是算角度的

## 6.霍夫变换

### 1.直线
检测直线分别标准霍夫变换`cv2.HoughLines()`和概率霍夫变换`cv2.HoughLinesP()`
电赛一般采用概率霍夫变换，省算力
用法是
`lines = cv2.HoughLinesP(edge, 1, np.pi/180, threshold, minLineLength, maxLineGap)`

- 第一个参数是输入图像，二值图（Canny较多）
- 第二个参数霍夫参数空间里\(rho\)每次增加多少，固定为1，表示1像素
- 第三个参数每次间隔1°，也固定
- 第四个参数是阈值，**参数空间里面累计投票数达到该值，才判定为一条直线**，越高越严格
- 第五个参数是可选项，表示线段的最小长度，小于这个长度不被判定为直线
- 第六个参数是可选项，表示同一直线上允许的最大空隙，如果两段线段在同一条无限长直线上，中  间断开的距离小于`maxLineGap`，就把两段合并成一条线段。
- 概率霍夫返回的是一个直线的起点和终点坐标，格式是(N, 1, 4)。(line\[0]=\[x1,y1,x2,y2])

原理：@

### 2. 霍夫圆
`circles = cv2.HoughCircles(gray, cv2.HOUGH_GRADIENT, dp, minDist, param1, param2, minRadius, maxRadius)`

- **gray**
    输入图像：必须是**单通道灰度图**，推荐提前高斯模糊降噪，不能直接用彩色图。
- **method**
    检测算法，目前 OpenCV 仅实现 `cv2.HOUGH_GRADIENT`，固定写死。
- **dp**
    累加器分辨率缩放比例，类比霍夫直线的 rho 步长。
    `dp=1`：累加器和原图分辨率一致，精度最高；
    `dp=2`：累加器长宽缩小一半，速度更快、精度下降。
    竞赛常规固定 `dp=1`。
- **minDist**
    检测出的两个圆心之间**最小像素距离**。
    如果画面两个圆靠得比这个值近，只会识别其中一个；防重复检出，赛道 / 靶标场景设 30~50。
- **param1**
    内部 Canny 边缘检测的**高阈值**，低阈值自动等于 `param1/2`。
    值越大，越难提取弱边缘，方块、杂点更容易被过滤；干净圆环设 80~100，弱对比度小圆设 50~60。
- **param2**
    **圆心投票阈值（核心参数，对应 HoughLinesP 的 threshold）**
    圆弧像素在参数空间投票，总票数超过该数值才判定为圆。
    数值越高，判定标准越严苛，只有完整、标准圆形才会被检出；方块、圆角矩形投票不足会直接过滤。
    空心圆环推荐 22~30，实心小圆 15~20。
- **minRadius（可选）**
    能识别的圆**最小半径**，小于该半径的小圆点直接忽略；过滤灰尘噪点。
- **maxRadius（可选）**
    能识别的圆**最大半径**，大于该半径的大圆直接忽略；过滤大块黑斑干扰。


`circles` 输出形状 `(1, N, 3)`
`circles[0][i] = [x_center, y_center, radius]`
- x_center：圆心横坐标
- y_center：圆心纵坐标
- radius：圆半径
无圆时 `circles = None`，代码必须做空判断，否则报错。

## 7. 直方图

直方图不只是"统计像素个数"——围绕它有一整套图像处理算法，从**统计→均衡→匹配→比较→反投影→分割**一条龙。  

### 6 类直方图算法

|类别|作用|典型 OpenCV API|
|---|---|---|
|**1. 计算与统计**|算像素分布、归一化、累积分布 CDF|`cv2.calcHist`、`cv2.normalize`|
|**2. 直方图均衡化**|拉伸对比度，让暗部细节出来|`cv2.equalizeHist`（全局）/ `cv2.createCLAHE`（局部）|
|**3. 直方图匹配 / 规定化**|把图 A 的色调调成图 B 的分布（白平衡/风格迁移）|手写累计分布映射|
|**4. 直方图比较**|两张图的相似度（目标检索、跟踪）|`cv2.compareHist`（Correlation / Chi-Square / Bhattacharyya）|
|**5. 直方图反投影**|在大图里找"跟目标直方图相似的区域"|`cv2.calcBackProject` + `CamShift`|
|**6. 基于直方图的分割**|Otsu、三角法等自动阈值|`cv2.threshold(..., THRESH_OTSU)`|

### 每一类的工程用途

#### ① 计算与统计

python

```
hist = cv2.calcHist([gray], [0], None, [256], [0, 256])  # 256 维灰度直方图
hist /= hist.sum()                                       # 归一化为概率
cdf = hist.cumsum()                                      # 累积分布
```

#### ② 均衡化（最常用）

python

```
# 全局：暴力拉伸，暗图细节拉出来
eq = cv2.equalizeHist(gray)

# CLAHE（局部 + 限对比度，推荐）：解决"一边亮一边暗"
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
cl = clahe.apply(gray)
```

#### ③ 直方图匹配（白平衡/风格化）

- 算 source 图的 CDF 和 target 图的 CDF
- 把 source 每个灰度值映射到"累积概率最接近"的 target 灰度值
- OpenCV 没现成 API，需要手写 8 行左右

#### ④ 直方图比较

python

```
cv2.compareHist(hist1, hist2, cv2.HISTCMP_CORREL)    # 相关
cv2.compareHist(hist1, hist2, cv2.HISTCMP_CHISQR)    # 卡方
cv2.compareHist(hist1, hist2, cv2.HISTCMP_BHATTACHARYYA)  # 巴氏
# 值越大（CORREL）或越小（CHISQR/BHATTACHARYYA）→ 越相似
```

#### ⑤ 反投影（跟踪、找目标）

python

```
# 输入：目标 HSV 直方图；输出：目标概率图
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
mask = cv2.inRange(hsv, lower_hsv, upper_hsv)        # 可选：只取 H+S 通道
prob = cv2.calcBackProject([hsv], [0, 1], roi_hist, [0, 180, 0, 256], scale=1)
# 经典组合：CAMShift 算法用 prob 做迭代收敛
```

#### ⑥ Otsu 等阈值分割

python

```
# Otsu 自动找最佳分割阈值（基于类间方差最大化）
_, th = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
```

### 工程实战用得最多的 3 个

|场景|用法|
|---|---|
|**电赛光照不均**|**CLAHE**（直方图均衡化变种）|
|**HSV 颜色阈值不够稳时**|直方图反投影 + CAMShift 跟踪|
|**多张图相似度排序 / 图像检索**|`compareHist` 用 Bhattacharyya 距离|

### 一句话总结

直方图算法 = **统计 + 均衡（CLAHE 是最强变种） + 匹配 + 比较 + 反投影 + 分割**——电赛现场用得最频繁的是 **CLAHE 做光照增强**、**calcBackProject + CAMShift 做目标跟踪**、**Otsu 做自动阈值**，这三招基本能覆盖 80% 的视觉调试场景。
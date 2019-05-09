Three.js太阳系运转
===================================
[Three.js](http://threejs.org)
----------------------------------- 


1. npm install
2. npm run dev
3. open [127.0.0.1:8080/solar/SolarSystem.html](127.0.0.1:8080/solar/SolarSystem.html)

### all Plants
init.js  ------------------初始化Three.js组件
data.js  ------------------太阳系原始数据


太阳系模型分析


八大行星的公转轨道半径 = 太阳的半径 + 日距
八大行星的公转轨道圆心偏转值 = (远日距 - 近日距)/2

太阳系最大范围 =  太阳半径 + 海王星日距

## 数据说明
首先是八大行星与太阳的距离远近排列顺序情况。八大行星离太阳的远近排列顺序为水星、金星、地球、火星、木星、土星、天王星、海王星。

　　其次是八大行星的体积大小排列顺序情况。如我们把地球的体积设为1，则太阳与八大行星之间的体积比例为：太阳：木星：土星：天王星：海王星：地球：金星：火星：水星＝1300000：1317：745：65：57：1：0.86：0.15：0.056，由此可以得出，木星是八大行星体积最大的，土星次之。

　　再来就是八大行星的质量大小排列顺序情况。行星的质量是评定是否是八大行星的条件之一，若将八大行星的重量及平均密度，从大到小做一个排序，则质量从大到小依次为：木星、土星、海王星、天王星、地球、金星、火星、水星。

　　最后是八大行星的亮度排列顺序情况。星星的明暗程度，在天文学上一律用星等来表示，记为m（即magnitude）。天文学上规定，星等数越小，说明星越亮，星等数每相差1，星的亮度大约相差2.5倍。

　　那么八大行星哪颗最亮眼呢？这需要依据情况而定，不同时期行星的亮度是不一样的，一般人们是根据这八大行星最亮的时候来判定。金星在最亮的时候，星等为－4.4m；而水星最亮的时候为－2.6m。火星最亮的时候，星等为－2.9112m；而木星最亮的时候星等达到－2.9467m。土星最亮的时候，星等为－0.3m；天王星最亮的时候，星等为5.5m；而海王星最亮的时候，星等达到7.8m。


### 准备
首先写一个简单的页面，准备一个canvas容器:
在js文件中准备好三要素：renderer, camera, scene

### 第一视觉移动
firstPersonControls通过距离（鼠标移动过的屏幕距离）和时间（通过Clock计算）计算而得出镜头视觉改变的速度，
相当于我们站着不动，转动眼睛、头部的视觉改变方式。
而镜头本身位置的改变相当于人走路，通过按键监听实现。

### 星球位置
我们需要让星球环绕着太阳（原点）做圆周运动，
在只需要考虑平面(x, 0, z)的情况下，
实际就是通过三角函数去计算星球的平面位置。
![](https://pic3.zhimg.com/80/c4fa526c3b503a2124a479cfd0716cbe_hd.png)

我们设置(0, 0)为太阳中心位置，P点(x, y)则为星体位置。y值会在实际中作为z的值。

我们给星体设置一个公转的角速度, 
每次animationFrame的执行中,我们都为星体累加角度, 
通过Math.sin(), Math.cos()即可顺利计算出星体当前的位置。

### 星球运动
我们要给每一个星体加上当前角度和角速度的属性。
当角度已经累加到2PI时，此时星体已经走过一圈了，所以可以把无用的2PI去掉。
由于动画大概每秒60帧，所以每秒钟大概会累加60*speed。
这个方法在move()中为每一个星体都执行一次，就可以真正动起来了。

### 运动轨迹辅助线
使用RingGeometry来制作运动轨迹，由于Ring默认是垂直于x轴，需要让它进行一次rotate。

### 关于光照
在这个太阳系的环境中，我们需要用到环境光和点光。
PointLight用来发射点光源，把它放在太阳的中心来模拟太阳发出的亮光，照亮所有的太阳系。
而行星的背面由于不会被太阳光照到，需要环境光AmbientLight来辅助照明。

## 星星
BufferGeometry。BufferGeometry会保存这个Geometry所有的数据，
including vertex positions, face indices, normals, colors, UVs, 
and custom attributes within buffers，从而大量减少GPU的运算压力。

* 准备工作。每一个Float32Array数组的每三个成员来确定一个行星的信息（位置、颜色）。

* 要定义一颗星星的位置，首先我们要决定它能够出现的范围。
在这里我简单的设置为：以原点为中心，在边长为2000的立方体内不能出现星星，
边长为4000的立方体外不能出现，类似于一个空心的立方体。
太近的话容易被镜头撞上，太远了镜头也很难捕捉。

* 在这种情况下，x, y, z三个坐标必须至少有一个坐标是比gap(1000)要大的，
这样才能保证不会出现在内层。

* 接下来在for循环中为星星添上颜色。color.setRGB会把0~1的数字转化为0~255的rgb色，
这里我选择范围为0.5~1的随机值，因为亮色的星星会比较好看。

* 最后设置bufferGeometry, 并让它进行空间计算。
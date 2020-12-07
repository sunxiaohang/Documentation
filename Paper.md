# Paper
## Navigation
### Recast principle
##### Recast Navigation是一个导航工具集
- Recast 负责根据模型生成导航网格
- Detour 利用导航网格做寻路操作
- DetourCrowd 提供群体寻路行为的功能
#### Recast 过程
- 场景模型体素化
- 过滤出可行走表面
- 生成Region
- 生成Contour(边缘)
- 生成PolyMesh
- 生成Detailed mesh

##### 体素化
和GPU渲染管线的光栅化处理流程概念一样,都是将矢量的模型信息(三角形),转换为点阵信息(像素或者体素)
##### 过滤出可行走表面
根据那些表面顶部有足够可行走空间(根据预设参数确定),剔除不符合要求的体素数据,初步计算出行走表面
##### 生成Region
根据可行走表面,使用特定算法,将这些可行走表面切分为一个个尽量大的,连续的,不重叠的,中间没有洞的区域,成为Region.由于不重叠,也就不需要高度信息.有以下三种算法
- 分水岭算法(watershed partitioning) 效果好\处理慢
- Monotone partioning 最快,但是region细长
- Layer partitioning 两者折中,比较依赖初始数据
##### 生成 contour
- 根据体素化信息和region,首先构建出描绘Region的Detailed contours(精确轮廓),由于Detailed Contour以体素化为单位构建边缘的,因此是锯齿状的
- 将Detailed contours简化为Simplified Contours(简化轮廓),方便后面的三角形化

##### 生成PolyMesh
大多数算法的处理逻辑都是基于凸多边形,这一步将Simplified Contours切分为多个凸多边形(polygon),在polygon中,任意两个点在二维平面都是可以直线到达的,因此,polygon是detour的基本寻路单元
##### 生成Detailed Mesh
如果吧场景的拓扑结构看成一个无向图,其中每个polygon是一个顶点,那么polygon只是在拓扑结构上解决寻路问题,但是为了在具体寻路过程中让角色更加贴合地面行走,需要一些更精确的地形信息(比如高度),因此需要将polygon拆分为更贴近地面的Detailed mesh
#### Detoru 过程
- polyMesh粒度寻路,返回Polymesh数组
- detailedMesh粒度寻路,返回坐标点数组形成的路径
## Animation
#### Playable API
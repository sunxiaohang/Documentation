### Character Controller or rigidbody
控制器本身不会对力做出反应，也不会自动推开刚体，在OnControllerColliderHit()函数对与控制器碰撞的任何对象施加力
##### skinwidth 设置约为10% 
作为未防止抖动，太大容易卡住
##### stepOffset 仅当角色比指示值更接近地面时，角色才会升高一个台阶

#### Jonit
##### character joint
- connectBody 关联对象
- Anchor 关联对象局部空间旋转点
- axis 旋转轴
-  connected anchor 手动配置旋转点
-  swing axis 摆动轴
-  high/low twist limit 关节上下限
-  break force 破坏力 （位移）
-  break torque 破坏扭矩 （旋转）
-  enable collision 允许关节间碰撞
##### extension
- bounciness 关节反弹力
- spring 关节关联物体的弹力
- damper 阻尼力
- contact distance 保持接触的距离，避免发生抖动

#### configurable joint
- lock轴 不允许任何移动
- limited轴 限制移动
- free轴 无限制

-弹簧属性 overmove会拉回
- 对于一些重复机械运动可以直接给关键施加驱动力slerp drive

#### fixed force
- force 世界空间受到的矢量力
- relative force 对象局部空间中应用的矢量力
- torque 世界空间的矢量扭矩
- relative torque 局部空间应用的矢量扭矩
##### sample
- 要使对象向上运动，添加y轴方向的恒定力
- 要使对象向前运动，添加z轴relative force的恒定力
##### fixed joint
将对象的移动限制为依赖另一个对象
##### hinge joint
将两个刚体组合在一起，对刚体进行约束，模拟链条，摆钟，门
- use motor 电机使对象旋转
  - target velocity 对象试图获得的速度
  - force 为获得该速度施加的力
  - free spin不会使用点击来制动旋转，置灰进行加速
- 可已将多个铰链官阶串在一起形成链条，为链条中每个链接添加一个关节，并将洗一个链接作为连接体
- spring合motor是互斥的，不可同时使用这两个属性
##### spring joint
将两个刚体联结在一起，但允许两者之间距离的改变，类似于弹簧链接
- 为防止弹簧无休止震荡，可以修改damper的值(根据两个对象之间的相对速度按照比例减小弹簧力,值越高震荡小时的越快)

### rigidbody
可以理解为物理agent，可以接受力和扭矩NVIDIA physX
- freeze position 停止(刚体)沿\[世界\]x,y,z轴的移动
- fresze rotation 定制(刚体)绕\[局部\]x,y,z轴的移动
- 图形学的移动和物理学的移动是互斥的
#### tips
- 是一个刚体有比另一个刚体更大的质量，并且不会使其在自由落体是降落的更快，可以使用drag
- 如果要通过变换组件移动游戏对象，但希望接受collision/tirgger消息，必须将刚体附加到正在移动的对象上
#### rigidbody运动
- addForce
- addTorque
- isKinematic通过图形学移动控制，本身不会受物理引擎的影响，但会影响其他刚体
##### rigidbody 父子化
当对象处于物理控制下时，对象的移动方式在一定程度上独立于其变换父对象的移动方式，如果移动任何父对象，会将刚体子对象拉回自己身边，但子对象刚体仍会因重力下落

### collider
#### mesh collider
- 标记为convex(凸面)的mesh collider可与其他meshcollider发生碰撞(最多255个三角形)
- IsTrigger 启用碰撞事件，关闭引擎碰撞
- 网格碰撞体通常不能(相互)碰撞(需要勾选Convex),典型的解决方案是所有启动对象使用原始碰撞体(降低性能开销),静态对象昂使用网格碰撞体

#### wheel collider
用于地面交通工具的特殊碰撞体，内置了碰撞检测，车轮物理组件和基于打滑的轮胎摩擦模型

#### continuous collider
连续碰撞检测的目的是作为一种安全机制，可以在对象会相互穿过的情况下捕获碰撞，但不会提供精确物理碰撞的结果(文档不全，应该是指受力)

#### time scaler
timer scaler 不会降低执行速度，只会更改通过Time.deltaTime和time.fixedDeltaTime报告给update和fixedUpdate函数的时间步长，当游戏时间减慢时，调用update的频率会升高，Time.deltaTime会缩短，其他脚本函数不受时间标度影响

#### capture frame
录制游戏过程为视频
```
public class ExampleScript : MonoBehaviour {
    string folder ="ScreenshotFolder";
    int frameRate = 25;
    void Start () {
        // 设置回放帧率（实时时间将与此后的游戏时间不相关）。
        Time.captureFramerate = frameRate;
    }
    void Update () {
        string name = string.Format("{0}/{1:D04} shot.png", folder, Time.frameCount );
        Application.CaptureScreenshot(name);
    }
}
```
##### coroutine
coroutine会在update函数返回后运行
### Unity event
借助unityevent可以让用户驱动的回调从编辑时间一直持续到运行时
- 内容驱动的回调
- 解耦系统
- 持久回调
- 预配置的调用事件
- tips:unityEvent于标准位图哦有类似的限制，保留引用导致垃圾回收无法进行(注意释放)

#### UI消息系统
ui系统使用一种消息系统来取代sendMessage
```
//自定义消息
public interface ICustomMessageTarget:IEventSystemHandler{
    void Message1();
    void Message2();
}
//发送消息
所有实现接口的组件上执行Message1函数
ExecuteEvents.Execute<ICustomeMessageTarget>(target,null,(x,y)=>x.Message1());
```
### graph
#### 光照方式
- 实时光照
- 烘培光照贴图
- 预计算实时全局光照

##### scene窗口
- skybox material 天空盒是一种材质，出现在场景中所有对象的后方，模拟天空或遥远的背景
- sun source 设置太阳，如果none(默认)，假设场景中最靓的(方向光)代表太阳
- source 漫反射环境光
  - color 选择颜色作为环境光
  - gradient 来自天空，地平线，环境光设置单独颜色，并在他们之间混合
  - skybox 天空盒
intensity multuplier 设置场景中漫反射光的亮度(1-8)默认1
##### 光源类型
- 点光源
- 聚光灯
- 方向光
- 面光源
- 发光材质smission
- 环境光

##### linght inspector
- type 光源类型
- intensity 强度
- indirect multiplier 改变间接光强度(直射光反射后的强度)\
- cookie 指定用于投射阴影的纹理遮罩
- draw halo 回值直径定于range的球形光环

#### camera
相机视口时经过标准化的(0,0)->(1,1)中心点为(0.5,0.5)
高depth相机画面会覆盖低depth相机画面
- camera.enable = true/false切换相机
- 设置相机的viewport rect属性改变相机在屏幕上的矩形大小
##### physical camera
模拟真实相机
- focal length 传感器和摄像机镜头之间的距离，即焦距，此属性决定垂直视野
- sensor size 捕捉图像的传感器的宽度和高度，决定了物理相机的宽高比
- length shift 旋转相机拍摄较大物体会使水平线汇聚，而水平移动镜头会重新框定图像且保持水平线为直线

##### 相机参数计算公式
float frustumHeight = 2.0f*distance*Mathf.Tan(camera.fieldOfView*.5f*Mathf.Deg2Rad);
float distance = frustumHeight/2/Mathf.tan(camera.fieldOfView*.5f*Mathf.Def2Rad);

float frustumHEight = frustumWidth/camera.aspect;
##### 推拉表变焦
```
float FrustumHeightAtDistance(float distance){
    return 2.0f * distance * Mathf.Tan(camera.fieldOfView * 0.5f * Mathf.Deg2Rad);
}
float FOVForHeightAndDistance(float height, float distance){
    return 2.0f * Mathf.Atan(height * 0.5f / distance) * Mathf.Rad2Deg;
}
void Start(){
    float distance = Vector3.Distance(camera.transform.position,target.position);
    initHeightAtDist = FrustumHeightAtDistance(distance);
}
void Update(){
    transform.Translate(Input.GetAxis("Vertical") * Vector3.forward * Time.deltaTime * 20);
    float currentDistance = Vector3.Distance(camera.transform.position,target.position);
    camera.fieldOfView = FOVForHeightAndDistance(initHeightAtDist,currentDistance);
}
```
#### 相机投射
- ScreenPointToRay 期望以像素坐标的形式提供该点
- ViewportPointToRay 接受标准化的(0,0)>(1,1)
##### 沿着射线移动相机
例如允许用户使用鼠标选择对象，然后将对象放大，同时将其固定到鼠标下的相同屏幕位置
```
Ray ray = camera.ScreenPointToRay(Input.mousePosition);
float zoomDistance = zoomSpeed * Input.GetAxis("Vertical") * Time.deltaTime;
camera.transform.Translate(ray.direction*zoomDistance,Space.World);
```
##### 使用斜视锥体
```
Matrix4x4 mat = Camera.main.projectionMatrix;
mat[0, 2] = horiObl;//[-1,1]水平方向偏移
mat[1, 2] = vertObl;//[-1,1]垂直方向偏移
Camera.main.projectionMatrix = mat;
```
##### 遮挡剔除和视锥体剔除
遮挡剔除的数据由单元格组成，每个单元格是从整个场景包围体上细分而来，形成一个二叉树。
遮挡剔除使用两个树，
- 一个用于视图单元格(静态对象) > 映射到一个定义可见静态对象的索引列表
- 一个用于目标单元格(移动对象)

### timeline
timeline会保存时间轴资源和时间轴实例
- 时间轴资源 存储轨道，剪辑和录制动画，不保存动画的特定游戏对象的链接
- 时间轴实例 存储由时间轴资源动画化的特定游戏对象的链接，链接称为绑定，将保存到场景
尽管时间轴资源定义了过场动画，影片或游戏序列的轨道和剪辑，但无法将时间轴资源直接添加到场景，要使用时间轴资源对场景中的游戏对象进行动画化，必须创建时间轴实例
时间轴资源和时间轴实例是分开的，要重复使用时间轴资源可以为其他游戏对象创建新的时间轴实例
- 轨道的渲染和动画优先级为自上而下，对轨道进行重新排序即可更改轨道的渲染或动画优先级
#### playealbe director组件


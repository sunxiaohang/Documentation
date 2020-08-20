## Shader
#### 应用阶段
应用主导,通常由CPU负责实现（开发者具有绝对控制权）
- 准备场景数据
- 优化(粗粒度剔除(culling)工作,剔除不可见物体)
- 设置好每个模型的渲染状态(材质，纹理，使用的shader等)
- 输出是渲染所需的几何信息，即渲染图元(rendering primitives)
#### 几何阶段
几何阶段用于处理所有要绘制的几何相关（图元是什么，怎么绘制，在哪绘制）
负责和每个渲染图元交互，进行逐顶点、逐多边形的操作。这一阶段会将输出屏幕空间的二位顶点坐标，每个定点对应的深度值，着色等相关信息，并传递给下一阶段
#### 光栅化阶段
使用上个阶段产生的数据产生屏幕上的像素，并渲染出最终的图像，这一阶段实在GPU上执行（决定每个渲染图元上的哪些元素被绘制到屏幕上）
### CPU和GPU之间的通信
#### 应用阶段
- 把数据加载到显存（VideoRAM）[RAM=>VRAM]{GPU无法直接访问RAM}
- 设置渲染状态（场景中的网格怎么被渲染[使用哪个顶点着色器(Vertex Shader)/片元着色器(Fragment Shader),光源属性，材质等]）
- 调用Draw Call（CPU=>GPU）指令仅仅会指向一个需要被渲染的图元(primitives)列表（材质信息已在上一阶段处理完成）
顶点着色器(Vertex Shader): -完全可编程-，通常用于实现顶点的空间变换、顶点着色等
顶点着色器输入来自CPU，处理单位是顶点（输入进来的每个顶点都会调用一次顶点着色器）【顶点之间是相互独立的可并行】
- 坐标变换：把顶点坐标从模型空间转换到齐次裁剪空间
- 计算和输出顶点颜色：逐顶点光照
曲面细分着色器(Tessellation Shader):用于细分图元
几何着色器(Geometry Shader):可以被用于执行逐图元(Per-Primitive)的着色操作，或被用于产生更多的图元
裁剪(Clipping):裁剪掉不在摄像机视野内的顶点，并剔除掉某些三角图元的面片（归一化的设备坐标[Normalized Device Coordinates]）
一个图元和摄像机视野的三种关系：1.完全在视野内（继续传递给下一阶段） 2.部分在视野内（裁剪） 3.完全在视野外（不传递）
屏幕映射:把每个图元的x和y坐标转换到屏幕坐标系
- OpenGL：零点在左下角
- DirectX：零点在左上角
三角形设置：会计算光栅化一个三角形网格所需的信息（为了能够计算边界像素的坐标信息，我们需要得到三角形边界的表示方式）
三角形遍历：检查每个像素是否被一个三角网格覆盖，如果覆盖就生成一个--片元(fragment)--也被称为扫描变换
片元着色器(Fragment Shader):实现逐片元的着色操作
根据从顶点着色器输出的数据插值得到一个或多个颜色值
逐片元操作(Per-Fragment Operations):负责很多重要操作，例如修改颜色，深度缓冲，混合等（逐片元合并）
- 决定每个片元的可见性，涉及很多测试工作，如深度测试，模板测试等
- 如果一个片元通过了所有测试，仅需要把这个片元的颜色值和已经存储的颜色缓冲区中的颜色进行合并（混色）
#### Q&A
问题1:CPU和GPU是怎样实现并行工作的
--命令缓冲区(Command Buffer)--：包含一个命令队列，CPU添加命令，GPU读取命令
问题2:为什么DrawCall多了会影响帧率
--每一个DrawCall都需要很多额外的操作--
问题3:如何减少DrawCall
--批处理(Batching)--：把大量小的DrawCall合并为大的DrawCall
• 避免使用大量很小的网格（不可避免时考虑合并）
• 避免使用过多材质（尽量在不同网格之间公用同一个材质）
固定管线渲染
固定函数的流水线（Fixed-Function Pipeline）:提供一系列配置接口，开发者没有对流水线的完全控制权
#### Unity Shader(Unity Shader&Material)
• 创建一个材质
• 创建一个Unity Shader，并把它赋给上一步创建的材质
• 把材质赋给需要渲染的对象
• 在材质面盘调整Unity Shader属性得到免疫的效果
Material：材质需要结合一个GameObject的Mesh或者Particle System组件工作，它决定游戏对象的样子
```shader
Shader "shader路径/Shader名"
{
    Properties
    {
        //属性
        Name("display name",PropertyType)=DefaultValue;
        //PropertyType====Int,Float,Range(min,max),Color,Vector,2D,Cube,3D
        //实例
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }
    SubShader
    {
        //-------------显卡A使用的子着色器
        //真正意义上的shader代码
        //Surface Shader/ Vertex Shader / Fragment Shader / Fixed Function Shader
        //表面着色器/顶点着色器/片元着色器/固定函数着色器
        //标签设置 [Tags]
        Tags { "RenderType"="Opaque" }
        //状态[RenderSetup]
        
        //Pass 每个Pass定义了一次完整的渲染流程(Pass过多会造成渲染性能的下降)
        Pass{}
    }
    FallBack "Diffuse"
}
```
#### 状态设置
--定义在SubShader中会应用到所有Pass，定义在单独Pass中仅作用与单个Pass--
- -Cull-: -Cull Back- | -Front- | -Off- 设置剔除模式：剔除背面/正面/关闭剔除
- -ZTest-: ZTest Less Greater | LEqual | Equal | NotEqual | Always 设置深度测试时使用的函数
- -ZWrite-: ZWrite On | Off 开启/关闭深度写入
- -Blend-: Blend SrcFactor DstFactor开启并设置混合模式
```
SubShader的标签
Tags{"TageName"="Value" "TagName"="Value"}
//Queue:控制渲染顺序，指定该物体属于哪一个渲染队列，这种凡是可以保证所有的透明物体可以在所有不透明物体后被渲染
Tags{"Queue"="Transparent"}
//RenderType:对着色器进行分类，例如透明着色器和不透明着色器
Tags{"RenderType"="Opaque"}
//DisableBatching:一些SubShader在使用Unity的批处理功能时会出现问题
Tags{"DsiableBatching"="True"}
//ForceNoShadowCasting:控制使用该SubShader的物体是否会投射阴影
Tags{"ForceNoShadowCasting"="True"}
//IgnoreProjector:如果该标签为True，那么使用该SubShader的物体将不会受Projector的影响，通常用于半透明物体
Tags{"IgnoreProjector"="True"}
//CanUseSpriteAtlas:当该标签适用于精灵(sprites)时，将该标签设置为false
Tags{"CanUseSpriteAtlas"="False"}
//PreviewType:知名材质面板将如何预览该材质，默认情况下，材质显示为一个sphere，可设置为Plane，SkyBox
Tags{"PreviewType"="Plane"}
- Queue:控制渲染顺序，指定该物体属于哪一个渲染队列，这种凡是可以保证所有的透明物体可以在所有不透明物体后被渲染=》Tags{"Q"}
```
#### Pass语义块
//Pass的使用
UsePass "MyShader/PASSNAME"//Pass的名字会被转换为全部大写
```
Pass{
    //[Name]
    Name "PassName"
    //可以在Pass里面通过UsePass复用其他Pass
    UsePass "PassPath/PASSNAME"
    //GrabPass:该Pass负责抓取屏幕并将结果存储在一张纹理中，用于后续的Pass处理
    //[Tags]=>Tags{"tagName"="tagValue"}
    Tags{"LightMode"="ForwardBase"}
    Tags{"RequireOptions"="SoftVegetation"}
    //[RenderSetup]
}
Surface Shader表面着色器（无Pass）
Shader "Custom/CustomTestShader"
{
    Properties{}
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        CGPROGRAM
        //表面着色器的特定语法放在CGPROGRAM和ENDCG之间
        //CG/HLSL编写
        ENDCG
    }
    FallBack "Diffuse"
}
Vertex / Fragment Shader顶点/片元着色器（Pass内）
Shader "Custom/CustomVertexShader"
{
    SubShader
    {
        Pass
        {
            CGPROGRAM
            float4 vert(float4 v:POSITION):SV_POSITION
            {return UnityObjectToClipPos(V);}
            float4 frag() : SV_Target
            {return fixed4(1.0,0.0,0.0,1.0);}
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```
#### 着色器选择
- 兼容旧设备(Fixed Function Shader)
- 光源相关(Surface Shader)，移动平台表现不佳
- 使用光源较少||自定义渲染效果较多(Vertex/Fragment Shader)
坐标系
- 左手坐标系=》旋转{左手法则}
- 右手坐标系=》旋转{右手法则}
在模型空间和世界空间，Unity使用的是左手坐标系
在观察者空间(摄像机)，Unity使用的是右手坐标系，摄像机的前向是z轴的负方向
模型空间(对象空间)(局部空间)坐标系
世界空间
观察者空间(右手坐标系)
裁剪空间(齐次裁剪空间)空间是由--视锥体(View Frustum)--确定
视锥体有两种投影类型
- 1.正交投影(orthographic projection)
- 正交投影网格大小相同(保留了物体的距离和角度)[2D游戏或渲染小地图]
- 正交投影近裁剪平面和远裁剪平面大小相等
- 2.透视投影(perspective projection)
- 透视投影网格近大远小(模拟了人眼看世界的方式)[模拟3D世界物体]
- 透视投影近裁剪平面和远裁剪平面近小远大

#### 标准shader模板
```
Shader "Custom/Shader"
{
    Properties{_VariableName("Inspector name",valueType)= defaultValue;}
    SubShader
    {
 Tags{"TagName" = "TagValue"}  
 Pass
 {
    CGPROGRAM
    #pragma vertex vert
    #pragma fragment frag
    #include "UnityCG.cginc"
    struct appdata_base{};
    struct v2f{};
    v2f vert(appdata v){//vertex action}
    float4 frag(v2f i):SV_Target{//fragment action}
    ENDCG
 }
    }
}
```
#### Shader 分类
- 标准着色器
- 透明着色器
- 镂空着色器
- 自发光着色器
- 反射着色器
##### SubShader
- Queue 定义渲染顺序
- Background 值为1000，比如天空盒子
- Geometry 值为2000，大部分物体，包括不透明物体
- AlphaTest 值为2450 已进入AlphaTest的物体
- Transparent 值为3000 透明物体
- Overlay 值为4000 镜头光晕等
- 用户可以定义任意值 如"Queue"="Geometry+10"
- RenderType unity可以运行时替换符合特定renderType的所有shader，Camera.renderWithShader或者camera.setReplacementShader配合使用
- Opaque 绝大部分不透明物体
- Transparent 绝大部分透明物体，包括粒子特效
- Background 天空盒子
- Overlay GUI，镜头光效
- 参考rendering with replace shaders 或自定义
- ForceNoShadowCasting 为true时表示不接受阴影
- IgnoreProjector 值为true时表示不接受projector组件的投影
Pass 的 tag
- LightMode 指定pass和unity的哪一种渲染路径
- ForwardBase 在前向渲染中使用 只受环境光，主要的方向光，球协光照和lightMap影响
- ForwardAdd 在前向渲染中使用
- Deferred 在延迟渲染中使用，渲染到GBuffer
- Always 永远渲染，不处理光照
- ShadowCaster 用于渲染产生阴影的物体
- ShadowCollector 用于手机物体阴影到屏幕坐标buff里
- saturate 指饱和处理，大于1就是1，小于0就是0
#### Unity Shader 内置变量
模型空间(M>>)世界空间(V>>)观察空间(P>>)裁剪空间
- UNITY_MATRIX_MVP 当前的模型观察投影矩阵，用于将顶点/ 方向矢量从模型空间变换到裁剪空间
- UNITY_ MATRIX_MV 当前的模型观察矩阵， 用于将顶点/ 方向矢量从模型空间变换到观察空间
- UNITY_MATRIX_V 当前的观察矩阵， 用于 将 顶点/ 方向 矢量 从世界空间 变换 到 观察空间
- UNITY_MATRIX_P 当前的投影矩阵， 用于 将 顶点/ 方向 矢量 从观察空间 变换 到 裁剪空间
- UNITY_MATRIX_VP 当前 的 观察 投影 矩阵， 用于 将 顶点/ 方向 矢量 从世界空间 变换 到 裁剪空间
- UNITY_ MATRIX_T_MV UNITY_ MATRIX_ MV 的 转置矩阵
- UNITY_ MATRIX_IT_MV UNITY_ MATRIX MV 的 逆转置矩阵， 用于将法线从模型空间变换 到观察空间， 也可用于得到UNITY MATRIX_ MV的 逆 矩阵
- _Object2World 当前 的 模型矩阵， 用于将 顶点/ 方向矢量 模型空间 变换到世界空间
- _World2Object _Object2World 的 逆矩阵, 用于将顶点/方向矢量从世界空间变换到模型空间
#### Camera和屏幕参数
- _WorldSpaceCameraPos float3 该 摄像机 在世 界 空间 中的 位置 _
- _ProjectionParams float4 x = 1. 0（ 或 − 1. 0， 如果 正在 使用 一个 翻转 的 投影 矩阵 进行 渲染）， y = Near， z = Far， w = 1. 0 + 1. 0/ Far， 其中 Near 和 Far 分 别是 近 裁剪 平面 和 远 裁剪 平面 和 摄像机 的 距离
- _ScreenParams float4 x = width， y = height， z = 1. 0 + 1. 0/ width， w = 1. 0 + 1. 0/ height， 其中 width 和 height 分 别是 该 摄像机 的 渲染 目标（ render target） 的 像素 宽度 和 高度
- _ZBufferParams float4 x = 1 − Far/ Near， y = Far/ Near， z = x/ Far， w = y/ Far， 该 变量 用于 线性化 Z 缓存 中的 深度 值（ 可 参考 13. 1 节）
- unity_ OrthoParams float4 x = width， y = heigth， z 没有 定义， w = 1. 0（ 该 摄像机 是 正交 摄像机） 或 w = 0. 0（ 该 摄像机 是 透视 摄像机）， 其中 width 和 height 是 正交 投影 摄像机 的 宽度 和 高度
- unity_ CameraProjection float4x4 该 摄像机 的 投影 矩阵
- unity_ CameraInvProjection float4x4 该 摄像机 的 投影 矩阵 的 逆 矩阵
- unity_ CameraWorldClipPlanes[ 6] float4 该 摄像机 的 6 个 裁剪 平面 在世 界 空间 下 的 等式， 按 如下 顺序： 左、 右、 下、 上、 近、 远 裁剪 平面
#### Unity中的屏幕坐标
```
 VPOS/WPOS:分别是HLSL和Cg对屏幕坐标的定义，两者在unity中等价
//VPOS/WPOS语义定义的输入是一个float4类型的变量
//xy值代表在屏幕空间中的像素坐标，[0.5-400.5]
//unity中，VPOS/WPOS的z分量范围是[0,1]，在摄像机近裁剪平面，z值为0，远裁剪平面，z为1
//w分量表示摄像机的投影类型{透视投影[1/Near,1/Far](Near和Far对应裁剪平面到摄像机的距离)}{正交投影[恒为1]}
//把屏幕空间初一屏幕分辨率来得到{视口空间(viewspace)}中的坐标
//视口坐标是把屏幕坐标归一化，左下角(0,0)右上角就是(1,1)[xy值除以屏幕分辨率即可]或者用ComputeScreenPos函数：
fixed4 frag(float4 sp:VPOS):SV_Target
{
    return fixed4(sp.xy/ScreenParams.xy,0.0,1.0);
}
//ComputeScreenPos函数使用示例
struct vertOut{
    float pos:SV_POSITION;
    float scrPos:TEXCOORD0;
}
vertOut vert(appdata_base v){
    vertOut o;
    o.pos = mul(UNITY_MATRIX_MVP,v.vertex);
    o.scrPos = ComputeScreenPos(o.pos);
    return 0;
}
fixed4 frag(vertOut i):SV_Target{
    float2 wcoord = (i.scrPos.xy/i.scrPos.w);
    return fixed3(wcoord,0.0,1.0);
}
```
### Unity Cg语法汇总
#### 额外基础类型
- half 16位浮点数 [-60000,60000]精度{小数点后3.3位}
- fixed 12位定点数[-2,2]，精度1/256
- sampler* 分为sampler,sampler1D,sampler2D,sampler3D,samplerCUBE和samplerRECT
Swizzle操作符（只能用于结构体和向量）
float4(a,b,c,d).xwz等价于 float(a,d,c)
输入数据关键字
- Varying：在Cg程序中通过语义进行绑定Cg语言提供了一组语义词，用以表示参数是由顶点的哪些数据初始化的，语义词绑定后=>变量被初始化的同时也以为这变量有了特殊含义，如表示位置、法线等{语义提供了一种使用随顶点变化或随片段变化的值来初始化Cg程序参数的方法，如：顶点位置，法向量，纹理坐标等}
- Uniform参数：Uniform是用来限制一个变量的初始值的来源，表示该变量初始值来源于外部环境，【Unity内置Uniform输入参数列表(部分如：模型矩阵，视锥体，时间量，相机位置，光源位置等)】
语义：表示图元数据的含义(顶点位置，法向量或者纹理信息)也表示这些图元数据存放的硬件资源，顶点着色器的输出即是片段着色器的输入（因此两者输出输入语义必须是一致的）[语义只对这两个阶段有意义，且只在入口函数有效，在内部函数无效]可以理解为输出>输入关系映射，语义词和语义绑定如下：
- POSITION/SV_POSITION：模型坐标位置{两者唯一区别是：当用在顶点着色器中+作为语义输出时，SV_POSITION表示的顶点位置会被固定，不能改变}
- TANGENT：正交于表面法线的向量
- NORMAL：表面法线向量，需要进行归一化
- TEXCOORDi：第i组纹理坐标(即UV坐标，范围在0-1之间)，i是0-7中的一个数字
- COLOR：颜色(光照计算公式=ambient+Diffuse+Specular)
- PSIZE：点的大小
- BLENDINGICES：通用属性，可以用它和TANGENT替换TEXCOORDi
顶点着色器的输出语义词：
- COLOR：颜色
- FOG：输入雾坐标
- PSIZE：大小
- POSITION：模型坐标位置
- TEXCOORD0-TEXCOORD7：第i组纹理坐标
片段着色器输入语义定义：
- COLOR：颜色
- DEPTH：片段深度
语义绑定三种方法
```
//方法一:绑定语义放在函数的参数列表的参数声明后[]：表示可选项<>：表示必选项
[const][in|out|inout|uniform]<type><identifier>[ :<binding-semantic>][=<initializer>]  
void vert(float4 obj_position:POSITION,float4 obj_normal:NORMAL,out float4 outPos:POSITION,uniform float4 uColor:COLOR){}
//方法二:绑定语义放在结构体的成员变量后
Struct<struct-tag>{
    <type><identifier>[:binding-semantic];
    //sample
    float4 pos : SV_POSITION;
    float4 col:TEXCOORD0;
}
//方法三:绑定语义放在函数生命后面
<type><identifier>(<parameter-list>)[:<binding-semantic>]{<body>}
float4 frag(vertexOutPut input):COLOR{
    float4 color = float4(1,0,0,1);
    return color;
}
```
#### 使用属性 
材质和Unity Shader是紧密联系的，材质提供给我们一个可以方便的调节Unity Shader中参数的方式，通过这些参数，我们可以随时调整材质的效果，这些参数需要写在Properties语义块中
```
Shader "UnityShader/ShaderName"{
    Properties{
        _Color("Color Tint",Color)=(1.0,1.0,1.0,1.0)
    }
    SubShader{
        Pass{
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //属性定义想要在Cg中访问，需要在Cg中定义变量（名字相同，类型为对应类型）
            fixed4 _Color;
            struct a2v{
                float4 vertex:POSITION;
                float3 normal:NORMAL;
                float4 texcoord:TEXCOORD0;
            };
            struct v2f{
                float4 pos:SV_POSITION;
                fixed3 color:COLOR0;
            }
            v2f vert(a2v v):SV_POSITION{
                v2f 0;
                o.pos=nul(UNITY_MATRIX_MVP,v.vertex);
                o.color = v.normal*0.5+fixed3(0.5,0.5,0.5);
                return o;
            }
            fixed4 frag(v2f i):SV_Target{
                fixed3 c = i.color;
                c*=_Color.rgb;
                return fixed4(c,1.0);
            }
            ENDCG
        }
    }
}
```
#### ShaderLab属性类型和Cg类型变量匹配关系
ShaderLab属性类型 Cg变量类型
- Color,Vector
- float4,half4,fixed4
- Range,Float
- float,half,fixed
- 2D
- sampler2D
- Cube
- samplerCube
- 3D
- sampler3D
#### Unity中的基础光照
渲染总是围绕着一个基础问题：我们如何决定一个像素的颜色？从宏观上说，渲染包含两大部分：决定一个像素的可见性，决定这个像素的光照计算
- 光照用l表示方向，用辐照度(irradiance)量化光照
- 光垂直照射物体表面，光线间距为d，斜向照射，照射到物体上的光线间距为d/cos&，辐照度小于直射(点积)
- 光线与物体相交会有两种结果：1.散射(scattering) 2.吸收(absorption)
- 散射只改变光线方向，不改变光线密度和颜色
- 吸收只改变光线的密度和颜色，不改变光线的方向
着色 ：根据材质属性，光源信息，使用一个等式去计算沿某个观察方向的出射度的过程，也成为光照模型
BRDF光照模型(Bidirectional Reflectance Distribution Function)： 
当已知光源位置和方向，视角方向时，当给定模型表面上的一个点时，BRDF包含了对该点外观的完成描述，
标准光照模型 ：把进入摄像机的光线分为四个部分，每个部分用不同方法计算它的贡献度
- 自发光emissive(不会照亮周围物体，而是自身发光)
- 直接进入相机，不需要进行反射Cemessive = Memissive
-  高光反射specular(镜面反射方向散射的辐射量)
- 是一种经验模型，并不完全符合现实世界表面法线n，视角方向v，光源方向l，反射方向r
- 反射方向r可以通过其他信息计算得出r = 2(n*l)n-l
- Cspecular = (Clight*Mspecular)*max(0,v,r)[Mgloss]  ---{Mgloss是材质的光泽度gloss，也称反照率}用与控制高光区域的“亮点”有多宽，Mgloss越大，亮点越小Mspecular是材质的高光反射颜色，用与控制材质对于高光反射的强度和颜色，Clight是光源的颜色和强度
- 漫反射diffuse(光线照射到物体表面回想每个方向散射多少辐射量)
- 漫反射光照符合兰伯特定律，反射光线的强度与表面法线和光源方向之间夹角的余弦值成正比
- {{{Phong模型}}}---Cdiffuse = (Clight*Mdiffuse)*max(0,n*l);//防止记过为负
- {{{Blinn模型}}}---Cspecular = (Clight*Mspecular)*max(0,n*h)[Mgloss]
- h = (v+l)/|v+l|
- 环境光(ambient)用与描述其他所有的间接光源
- 光线通常会在多个物体之间反射，最后进入相机，用环境光模拟间接光照
- Cambient = Gambient，场景中所有物体都使用这个环境光
#### shader property
```
//Float类型，下面对应变量可以用flaot,half,fixed  
_Name("Inspector Name", Float) = defaultValue  
//Float类型,可以用一个滑动条控制范围,下面对应变量可以用float,half,fixed  
_Name("Inspector Name", Range(min, max)) = defaultValue  
//颜色类型，下面对应变量可以用float4,half4,fixed4，如果是颜色，尽量fixed4  
_Name("Inspector Name", Color) = (defaultValue.r, defaultValue.g, defaultValue.b, defaultValue.a)  
//2D纹理类型,默认纹理可以为空，白，黑，灰，凹凸，下面对应变量sampler2D  
_Name("Inspector Name", 2D) = "" / "white" / "black" / "gray" / "bump"{options}  
//长方形纹理，非2次方大小的纹理，其同上  
_Name("Inspector Name", Rect) = "" / "white" / "black" / "gray" / "bump"{options}  
//立方体贴图CubeMap  
_Name("Inspector Name", Cube) = "" {options}  
//传递一个Vector4向量  
_Name("Inspector Name", Vector) = (defaultValue.x, defaultValue.y, defaultValue.z, defaultValue.w)
```
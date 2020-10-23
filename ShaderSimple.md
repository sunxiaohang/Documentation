## Unity shader
### 渲染流水线
应用阶段>>[输出渲染图元]>>几何阶段>>[输出屏幕空间]>>光栅化阶段

##### CPU和GPU之间的通信
把数据加载到缓存>>设置渲染状态>>调用drawcall

##### GPU流水线
顶点着色器>>[曲面细分着色器]>>[几何着色器]>>裁剪(裁剪不在相机视野，剔除某些图元的面片)>>屏幕映射>>三角设置>>三角遍历>>片元着色器>>逐片元操作
**完全控制的几个阶段**
顶点着色器>>[曲面细分着色器]>>[几何着色器]>>片元着色器
#### 光栅化阶段
三角设置>>三角遍历>>片元着色器>>逐片元操作
##### 逐片元操作
片元--->模板测试Stencil Test>>深度测试Depth Test>>混合--->颜色缓冲区
#### 问题Q&A
**CPU和GPU是如何实现并行工作的**
命令缓冲区
**如何减少draw call**
通过batching的方式
- 避免使用大量很小的网格
- 避免使用过多的材质

### Shader
##### Shader 实例
```C#
shader "Shader/CustomShader"{
    Properties{
        //properties
        _Color("Color",Color)=(1.0,1.0,1.0,1.0)
    }
    SubShader{
        Pass{
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            fixed4 _Color;
            struct a2v{//application 2 vertex
                float4 vertex:POSITION;
                float3 normal:NORMAL;
                float4 texcoord:TEXCOORD0;
            }
            struct v2f{
                float4 pos:SV_POSITION;//告诉unity，pos里面包含了顶点在裁剪空间中的位置信息
                fixed3 color:COLOR0;//用于存储颜色信息
            }
            v2f vert(a2v v):SV_POSITION{
                v2f o;
                o.pos = mul(UNITY_MATRIX_MVP,v.vertex);
                o.color = v.normal*0.5_fixed3(0.5,0.5,0.5);
                return o;
            }
            fixed4 frag(v2f i):SV_Target{
                return fixed4(i.color,1.0);
            }
            ENDCG
        }
    }
    Fallback "VertexLit"
}
```
##### HLSL语义
填充到变量的语义来自于使用该材质的MeshRender提供，在每帧drawcall的时候，MeshRender组件会把她负责渲染的模型数据发送给UnityShader。
##### 变量
- Color,Vector  ======>   float4,half4,fixed4
- Range,Float  ======>   float,half,fixed
- 2D  ======>   sampler2D
- Cube  ======>   samplerCube
- 3D  ======>   sampler3D
Tips:uniform关键字是Cg中修饰变量和参数的一种修饰词，仅仅用于提供一些关于该变量的初始值是如何指定和存储的相关信息
##### 内置的包含文件
- 以.cginc结尾
- #include "UnityCG.cginc"
##### UnityCG.cginc中常用函数
- float3 ObjSpaceViewDir(float4 v)//输入模型空间顶点位置，返回模型空间从该点到相机的观察方向
- float3 WorldSpaceViewDir(float4 v)//输入模型空间顶点，返回世界空间从该点到相机的观察方向
- -----------------------------------分割-----------------------------------------------
- float3 ObjSpaceLightDir(float4 v)//[前向渲染]输入模型空间顶点，返回模型空间从该点到光源的方向[未归一化]
- float3 WorldSpaceLightDir(float4 v)//[前向渲染]输入模型空间顶点，返回世界空间从该点到光源的方向[未归一化]
- -----------------------------------分割-----------------------------------------------
- float3 UnityObjectToWorldNormal(float3 dir)//把法线方向从模型空间转换到世界空间
- - -----------------------------------分割-----------------------------------------------
- float3 UnityObjectToWorldDir(float3 dir)//把方向矢量从模型空间转换到世界空间
- float3 UnityWorldToObjectDir(float3 dir)//把方向适量从世界空间变换到模型空间中
#### Cg/HLSL语义
- 语义实际上就是一个赋给Shader输入和输出的字符串，这个字符串表达该参数的含义
- Unity只支持部分语义
- 特别含义语义[特定用途]{方便对模型数据的传输}
- 非特别含义语义[可以随便用]
- SV_(system-value)
##### a2v语义
- POSITION(float4) : 模型空间中的顶点位置
- NORMAL(float3) : 顶点法线
- TANGENT(float4) : 定点切线
- TEXCOORDn(float2,float4) : 该顶点纹理坐标
- COLOR(fixed4,float4) : 顶点颜色
##### v2f语义
- SV_POSITION : 裁剪空间的顶点坐标[必须]
- COLOR0 : 通常用于第一组顶点颜色
- COLOR1 : 通常用于第二组顶点颜色
- TEXCOORD0-7 : 通常用于输出纹理坐标
- 返回 SV_Target : 输出值将会存储到渲染目标renderTarget

### Unity中的基础光照
- 渲染总是围绕{像素的可见性}{像素的光照}
- 用辐照度来量化光
- 光线由光源发出后通常会有两个结果{散射scattering}{吸收absorption}
- 散射到外部的成为反射，反射又分为{漫反射}{高光反射}
- 根据材质，光源等，使用一个等式去计算沿某个观察方向的出射度的过程成为光照模型

#### BRDF光照模型(Bidirectional Reflectance Distribution Function)
图形学第一定律：如果他看起来是对的，那么他就是对的

#### 标准光照模型
把进入相机的光线分为4个部分，每一部分用不同的方法计算贡献度
- 自发光(emissive):描述当给定一个方向上时，一个表面本身会向该方向发射多少辐射量
- 高光反射(specular):描述光线从光源照到表面时，完全镜面反射出多少辐射量
- 漫反射(diffuse):描述表面会向每个方向反射多少辐射量
- 环境光(ambient):描述其他间接光照

##### 环境光ambient
- 光线会在多个物体之间反射最终进入摄像机
- C<sub>ambient</sub> = g<sub>ambient</sub>

##### 自发光emissive
- 直接由光源发出进入相机
- C<sub>emissive</sub> = m<sub>emissive</sub>

##### 漫反射diffuse
- 反射式完全随机的，近似认为任何方向上都是一样的，但是入射光角度很重要
- 漫反射符合兰伯特定律(lambert's law)
- C<sub>diffuse</sub> = (c<sub>light</sub>.*m<sub>diffuse</sub>)max(0,n.*l)
- m<sub>diffuse</sub>:漫反射颜色
##### 高光反射specular
- 并不完全符合真实世界的高光反射现象
- 使用phong模型计算高光反射,需要n,l,r,v四个矢量
- 反射方向r = 2(n.*l)*n-l
- C<sub>specular</sub> = (C<sub>light</sub>.*m<sub>specular</sub>)*max(0,v.*r)<sup>m<sub>gloss</sub></sup>
- m<sub>gloss</sub>成为光泽度或反光度，用于控制高光区域的亮点有多宽，值越大，亮点越小
- m<sub>specular</sub>:高光反射颜色
- Blinn模型 h = (v+l)/|v+l|
- C<sub>specular</sub> = (C<sub>light</sub>.*m<sub>specular</sub>)*max(0,n.*h)<sup>m<sub>gloss</sub></sup>

#### 逐像素/逐顶点光照
- 片元着色器中>>>逐像素{Phong着色/Phong插值/法线插值着色技术}
- 顶点着色器中>>>逐顶点光照{高洛德着色}图元内部线性插值
- saturate(x)防止负值

##### 逐顶点光照
- UNITY_LIGHTMODEL_AMBIENT.xyz 环境光
- (float3x3)unity_WorldToObject 矢量从模型空间到世界空间
- normalize(_WorldSpaceLightPos0.xyz) 光照方向
- _LightColor0.rgb 光照颜色
- _Diffuse.rgb 漫反射颜色
- LightColor*DiffuseColor = 反射颜色
```
fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;//环境光
fixed3 worldNormal = normalize(mul(v.normal,(float3x3)unity_WorldToObject));//从模型空间转换normal到世界空间并归一化
fixed3 wordLight = normalize(_WorldSpaceLightPos0.xyz);
fixed3 light = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal,wordLight));//光照颜色 漫反射颜色
o.color = ambient + light;
```
##### 逐像素光照
```
v2f vert(a2v v){
    v2f o;
    o.pos = UnityObjectToClipPos(v.position);
    o.worldNormal = mul(v.normal,(float3x3)unity_WorldToObject);//从模型空间转换normal到世界空间并归一化
    return o;
}
///环境光 UNITY_LIGHTMODEL_AMBIENT.xyz
fixed4 frag(v2f i):SV_Target{
    fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;//环境光
    fixed3 worldNormal = normalize(i.worldNormal);
    fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);
    fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal,worldLight));//光照颜色 漫反射颜色
    fixed3 color = ambient + diffuse;
    return fixed4(color,1.0f);
}
```
##### 半兰伯特Blinn-phong光照模型
```
v2f vert(a2v v){
    v2f o;
    o.worldPos = UnityObjectToClipPos(v.position);
    o.worldNormal = mul(v.normal,(float3x3)unity_WorldToObject);
    return o;
}
fixed4 frag(v2f i):SV_Target{
    //-------------------------环境光-----------------------
    fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
    //-------------------------漫反射------------------------
    fixed3 worldNormal = normalize(i.worldNormal);
    fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);
    fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*(dot(worldNormal,worldLight)*0.5+0.5);
    //------------------------高光---------------------------
    fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz); 
    fixed3 halfDir = normalize(worldLight + viewDir); 
    fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow( max( 0, dot( worldNormal, halfDir)), _Gloss);
    fixed3 color = ambient + diffuse+specular;
    return fixed4(color,1.0f);
}
```
### SRP 
- 设定视锥体剔除，遮挡剔除，渲染顺序，渲染用到哪些shader，以及后处理效果等


1.定义CustomRenderPiplineAsset:RenderPipelineAsset

2.override IRenderPipeline => return new CustomRenderPipeline();

3.override CustomRenderPipeline的Render方法 逐个渲染相机

### render 流程
#### 设置相机属性
- SetupCameraProperties//相机属性
- CommandBuffer.ClearRenderTarget(camera.clearFlags);//用command buffer方式clear flag 层
#### 视锥体剔除&遮挡剔除
- ScriptableCullingParameters //CullResults.GetCullingParameters(camera, out cullingParameters)获取相机矩阵或手动设置剔除
- CullREsults.Cull(ref cullingParameters,context,ref cullResult)//获取一个需要被绘制的物体列表
#### 绘制对象
- DrawRendererSettings drawSettings = new DrawRendererSettings(camera,new ShaderPassName("SRPDefaultUnlit"));//设置绘制对象的相机和shader
- drawSettings.sorting.flags = SortFlags.CommonOpaque;//开始渲染之前渲染对象按前后距离及渲染条件排序
- FilterRenderersSettings filterSettings = new FilterRenderersSettings(true) {renderQueueRange = RenderQueueRange.opaque};//默认FilterRenderersSetting不包含任何内容 true表示包含所有内容
- context.DrawRenderers(cullResult.visibleRenderers,ref drawSettings,filterSettings);//执行绘制过程，参数以此为visibility，draw setting，filter setting
```C#
//---------------------------------------设置相机属性----------------------------------------
context.SetupCameraProperties(camera);
CameraClearFlags clearFlags = camera.clearFlags;
commandBuffer.ClearRenderTarget(
    (clearFlags&CameraClearFlags.Depth)!=0,
    (clearFlags&CameraClearFlags.Color)!=0,
    camera.backgroundColor);//是否清除深度信息|是否清除颜色|设置清除颜色
context.ExecuteCommandBuffer(commandBuffer);//执行command buffer
commandBuffer.Clear();
//------------------------------------- 相机视锥体剔除----------------------------------------
ScriptableCullingParameters cullingParameters;//找出可以剔除的内容需要我们跟踪多个相机设置和矩阵，可以手动设置，也可以委托给静态CullResults.GetCullingParameters方法
if (!CullResults.GetCullingParameters(camera, out cullingParameters))return;
CullResults.Cull(ref cullingParameters ,context,ref cullResult);//ref修饰的变量必须要有值
//---------------------------------------draw something -------------------------------------
//一旦知道可见内容，就可以继续渲染这些形状，通过DrawRenderers上下文cull.visibleRenderers作为参数来完成，并告诉他要使用哪些渲染器。
//此外还要提供绘图设置和过滤器设置，两者都是struct DrawRendererSettings和FilterRenderersSettings
//相机用来设置排序和剔除层，过程则用来控制使用哪个着色器过程进行渲染
var drawSettings = new DrawRendererSettings(
        camera,
        new ShaderPassName("SRPDefaultUnlit")
    );
//-----------------不透明-------------------
drawSettings.sorting.flags = SortFlags.CommonOpaque;//开始渲染之前渲染对象按前后距离及渲染条件排序
var filterSettings = new FilterRenderersSettings(true) {
    renderQueueRange = RenderQueueRange.opaque//将天空和之前的绘制限制为仅不透明的渲染器
};
context.DrawRenderers(//默认FilterRenderersSetting不包含任何内容 true表示包含所有内容
    cullResult.visibleRenderers,
    ref drawSettings,
    filterSettings
    );
//---------------------天空盒------------------
context.DrawSkybox(camera);//渲染天空盒
//---------------------透明--------------------
drawSettings.sorting.flags = SortFlags.CommonTransparent;//透明的渲染工作原理有所不同，他结合正在绘制的颜色和之前绘制的颜色，这需要从后到前的绘制
filterSettings.renderQueueRange = RenderQueueRange.transparent;
context.DrawRenderers(
        cullResult.visibleRenderers,ref drawSettings,filterSettings
    );
DrawDefaultPipeline(context,camera);
commandBuffer.EndSample("RenderCamera");//---------------层次化帧调试器
context.ExecuteCommandBuffer(commandBuffer);
commandBuffer.Clear();
//渲染顺序，不透明>>>天空盒>>>透明
context.Submit();//上下文命令时缓冲的，实际工作发生在submit函数执行
```
#### 标准的Unity渲染管线
- Background 1000：0-1499
- Geometry 2000：1500-2399
- AlphaTest 2450：2400-2699
- Transparent 3000：2700-3599
- Overlay 4000：3600-5000
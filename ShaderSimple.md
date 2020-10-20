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
#### SRP 
- 设定视锥体剔除，遮挡剔除，渲染顺序，渲染用到哪些shader，以及后处理效果等
1.定义CustomRenderPiplineAsset:RenderPipelineAsset
2.override IRenderPipeline => return new CustomRenderPipeline();
3.override CustomRenderPipeline的Render方法 逐个渲染相机
#### render 流程
- 设置相机属性
  - SetupCameraProperties//相机属性
  - CommandBuffer.ClearRenderTarget(camera.clearFlags);//用command buffer方式clear flag 层
- 视锥体剔除
  - ScriptableCullingParameters //CullResults.GetCullingParameters(camera, out cullingParameters)获取相机矩阵或手动设置剔除
  - CullREsults.Cull(ref cullingParameters,context,ref cullResult)
- 绘制对象
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
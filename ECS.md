# 概论
实体组件系统 Entity Component System (ECS) 是 Unity 面向数据技术栈的核心。顾名思义，ECS 包括三个组成部分

+ Entities(实体) 是大量充斥在你的游戏或软件的实体，或说是物体
+ Components(组件) 是与你的各种实体相关联的数据，这些数据由数据本身组织在一起，而不是由实体来组织（这种组织方式的区别是面向数据的设计与面向对象设计最根本的区别之一）
+ Systems(系统) 是将组件数据从当前状态转变为下一个状态的逻辑——例如，一个系统可以用来将当前所有正在移动的实体的位置更新为实体的速度与自上一帧开始的时间差的乘积

# ECS核心内容

## ECS相关概念

实体组件系统 Entity Component System (ECS) 架构将定义（Entities），数据（Components），行为（Systems）独立区分开。架构聚焦于数据。 系统通过读取由实体索引的组件数据流，将数据的状态从某种输入状态转变为某种输出状态。

下图展示了这三个基本部分是如何工作的   

![ECS三部分如何工作](https://raw.githubusercontent.com/MadWeedFall/UnityDocs/master/img/ECS_manual_translate/ECSBlockDiagram.png)     

在上图中系统读取位移和旋转组件，将它们相乘用于更新与它们关联的本地坐标组件。
实体A与实体B都有渲染器组件，而实体C没有，这并不影响系统，因为系统并不关注渲染器组件（你可以创建一个需要渲染器组件的系统，在这种情况下，系统会忽略实体C；或者你也可以创建一个排除带渲染器组件实体的系统，这样系统就会忽略实体A和实体B）

#### 原型 Archetypes
某种特定的组件类型的组合称作原型。例如，某种3D对象可以有一个代表世界位移的组件，一个代表线性移动的组件，一个代表旋转的组件，还有一个用于视觉展现的组件。这种3D对象的每一个实例都与一个单独实体相关联，但由于他们都有一系列相同的组件，这些对象实例都可以归类为同一个原型。

![ECS三部分如何工作](https://raw.githubusercontent.com/MadWeedFall/UnityDocs/master/img/ECS_manual_translate/ArchetypeDiagram.png)     

如上图所示，实体A和B都属于原型M，而实体C属于原型N。

可以通过添加或者删除组件来任意改变实体所属的原型。例如，如果删除B的渲染器组件，B就会归类到原型N中。

#### 内存块

实体的原型决定了实体的组件存储在哪里。ArchetypeChunk对象表示一整块内存，这块内存中存储的都是相同原型的实体。如果一块内存满了，新的内存块是按照相同原型的新建实体来分配的。 如果通过添加或删除组件来改变一个实体的原型，那么这个实体会被转移到别的内存块中。

这种组织方式在原型和内存块之间建立了一种一对多的关系。这同时意味着只需要查找现有的原型就可以找到所有持有特定组件组合的实体，而原型的数量相对所有实体来说少的多，通常原型数量很小，实体的数量会很大。

注意实体的组件不是按照某种特定顺序存储的。向某个原型添加实体时，实体会被放到该原型的第一个能存得下的内存块中。内存块是紧密排列的，当某个实体从原型中被删除掉，内存块中排在最后的实体中的组件会被移动到组件数组中新增的空槽中。

#### 实体查询

可以使用EntityQuery来确定系统需要处理哪些实体。一次实体查询会在现有的原型中查找符合条件的结果。可以设置以下查询条件
+ All -- 原型必须包含在All类别(category)中的所有组件类型
+ Any -- 原型必须至少包含一个在Any类别中的组件类型
+ None -- 原型必须不包含任何在None类别中的组件类型

一次实体查询会产出包含本次查询指定类型组件的内存块列表。可以使用IJobChunk对这些内存块中的组件直接进行遍历，IJobChunk是一种特殊的ECS事务。也可以使用包含隐式查询IJobForEach或 non-job for-each loop来遍历。

## 实体

实体是ECS架构的三个关键组成元素之一。实体代表了游戏或软件中各种各样的“物体”。实体既没有行为，也没有数据，它只是定义了哪些数据可以需要放到一起。系统提供行为，而组件则用于存储数据。

实体本质上就是ID。你可以认为实体是一种默认连名字都没有的超轻量级的GameObject。实体ID是固定的。实际上这些实体ID是存储组件引用或是其他实体引用的唯一固定的方式。

EntityManager管理在World（世界）中的所有实体。EntityManager维护一个实体列表，同时组织实体相关的数据用于优化性能表现。

虽然实体本身没有类型，但是多组实体可以通过关联的数据组件的类型来归类。当你创建实体并添加组件到实体时，EntityManager记录已有实体中的各种唯一的组件组合方式。这种唯一的组合方式被称为ArcheType（原型）。当添加组件到实体的时候，EntityManager会创建对应的EntityArchetype结构体。可以通过已有的各种EntityArchetype创建属于对应原型大的新实体。也可以预先创建一个EntityArchetype然后用它来创建实体。

#### 创建实体

通过Unity编辑器创建实体是最简单的创建实体的方法。可以在运行时将场景中的GameObject和Prefab转化成实体。当需要在游戏或软件中动态创建实体时，也可以通过创建生成系统通过一个job（事物）生成多个实体。当然，也可以通过使用EntityManager.CreateEntity方法来一次创建一个实体。

#### 使用EntityManager创建实体

使用EntityManager.CreateEntity方法来创建实体，所创建的实体会在和EntityManager所属的同一个World（世界）对象中被创建。

你可以通过下列方式一个一个的创建实体：

+ 使用CommonType对象数组创建带有组件的实体
+ 使用EntityArcheType创建带有组件的实体
+ 使用Instantiate，拷贝一个已存在的实体，包括该实体当前的数据
+ 创建一个不带组件的实体，然后在向其添加组件。（可以在实体创建后立即添加组件或在需要的时候添加额外的组件。）

你也可以通过下列方式一次创建多个实体：
    
+ 使用CreateEntity在一个NativeArray中填充属于相同原型的实体。
+ 使用Instantiate在一个NativeArray中填充已存在实体的拷贝，包括实体中当前的数据。
+ 使用CreateChunk显式创建多个包含指定数量特定原型的实体的内存块

#### 添加删除组件

当时实体创建完成后，如果添加或删除组件，受影响实体的原型和EntityManager必定会讲变更数据转移到一个新的内存块中，同时压缩原来内存块中的组件数组。

对实体的修改会造成结构上的改变——当添加或删除组件时改变了SharedComponentData的值，从而销毁了实体。这一操作不能在Job（事务）中进行，因为会造成事务处理的数据不可用。相反，你应当向EntityCommandBuffer中添加命令来实现这些修改，并且在事务执行完成后这个命令缓冲区才会执行。

EntityManager 对单个实体和NativeArray中的多个实体都提供了删除组件的方法。详见Components（组件）一章。

#### 遍历实体

遍历拥有相同一系列组件的实体，是ECS架构设计的核心内容，参见访问实体数据一章

#### World（世界）

一个World包含一个EntityManager和一系列ComponentSystems。你可以随意创建多个World对象。通常你会创建一个逻辑仿真World和一个用于渲染或展示的World

进入Play Mode的时候默认会创建一个World，工程中的各种可用的ComponentSystem对象都会填充到这个World中。不过你也可以取消默认World的创建，并通过全局宏定义将其替换成你自己的代码。

\#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_RUNTIME_WORLD 取消默认运行时世界的创建

\#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_EDITOR_WORLD 取消默认编辑器世界的创建

\#UNIT_DISABEL_AUTOMATIC_SYSTEM_BOOTSTRAP 同时取消上面两种默认世界的创建

+ 默认世界创建源码（参见代码文件：Packages/com.unity.entities/Unity.Entities.Hybrid/Injection/DefaultWorldInitialization.cs）
+ 自动启动入口点 （参见代码文件：Packages/com.unity.entities/Unity.Entities.Hybrid/Injection/AutomaticWorldBootstrap.cs)

## 组件
组件是ECS架构的三个关键组成元素之一。组件代表游戏或软件中的数据。实体本质上是用于编号组件集合的标记。系统提供行为。

具体来讲，ECS架构中的组件是有以下各种“特征接口”之一的结构

+ IComponentData —— 用于泛用组件（general purpose）和内存块组件（chunk components）
+ IBufferElementData —— 用于在实体中关联动态缓存（dynamic buffers）
+ ISharedComponentData —— 用于根据原型中的值分类或编组实体。详见共享组件数据（Shared Componet Data）
+ ISystemStateComponentData —— 用于在实体中关联系统相关的状态，并用于检测单个实体的创建和销毁。详见系统状态组件（System State Components）
+ ISharedSystemStateComponentData -—— 是共享数据和系统数据的结合。详见系统状态组件（System State Components）

EntityMananger将实体中独特的组件组合组织成原型。它将所有相同原型的实体的组件存储到被称为内存块的内存区域中。在指定内存块中的实体都具有相同的组件原型。

![ArcheTypeChunkDiagram](https://raw.githubusercontent.com/MadWeedFall/UnityDocs/master/img/ECS_manual_translate/ArchetypeChunkDiagram.png)

上图展示了组件数据如何根据原型存储在内存块中。共享组件和内存块组件不是这么处理的，因为它们是在内存块之外存储的，这些类型的组件的单一实例在所有可应用内存块中应用到所有的实体上。你也可以选择将动态缓存存储在内存块之外，通常在查询实体的时候可以像处理其他组件类型一样处理动态缓存。

## 泛用组件

Unity中的组件数据ComponentData（在标准的ECS定义中称为组件）是仅包含实体数据实例的结构。ComponentData不能有除访问结构内数据的工具方法外的其他方法。所有的游戏逻辑和行为都应当在系统中实现。如果按照老的Unity系统定义，这和老的Component类有点相似，不过ComponentData只能包含变量。

Unity ECS提供了一个叫做IComponent的接口用于实现你自己的代码。

#### ICompoentData

旧版的Unity组件（包括MonoBehaviour）是面向对象模式的类，其中包含数据和方法用于定义行为。IComponentData则是纯粹的ECS风格组件，这意味着组件不定义行为，组件仅仅是数据本身。IComponentData与其说是一个类倒不是如说是一个结构体，也就是说它默认是值拷贝而不是引用拷贝。你会经常用到如下代码模式来修改数据：

```c#

var transform = group.transform[index]; // Read

transform.heading = playerInput.move; // Modify
transform.position += deltaTime * playerInput.move * settings.playerMoveSpeed;

group.transform[index] = transform; // Write

```

IComponentData 结构不能包含托管对象的引用。因为所有的ComponentData的生命周期都在无垃圾回收检测的内存块内存中。

详见源码文件：/Packages/com.unity.entities/Unity.Entities/IComponentData.cs

## 共享组件数据

共享组件是一种特殊的数据组件，可以用来根据共享组件的特定值划分实体子类型（是通过实体所属原型划分之外的划分方法）。当向实体中添加共享组件的时候，EntityManager将所有拥有相同共享数据值的实体放进相同的内存块中。共享组件允许系统同时处理相似的实体。例如，共享组件Rendering.RenderMesh,作为Hybrid.renderting包中的一部分，定义了包括网格（mesh），材质（material），阴影表面（recieveShadows）等属性。在渲染过程中最高效的处理方式是将所有这些属性值相同的3D对象一起处理。因为这些属性被指定在一个共享组件中，EntityManager将相同的实体在内存中放到一块，所以渲染系统可以高效的遍历这些实体。

**注意**：过度使用共享组件可能导致内存块的利用率低下，因为这涉及到基于原型划分和各种共享组件的唯一属性值划分这两种方式结合所引发的内存块数量膨胀。可以通过使用Entity Debugger来查看当前的内存块利用情况来避免将不必要的属性添加到共享组件中。

如果你从实体中添加或者删除组件，或是改变了共享组件中的值，EntityManager会讲实体专移动到别的内存块中，必要时会创建新的内存块。

IComponentData 对于彼此间数据不同的实体来说通常是适用的，例如存储世界位置，表示碰撞点，例子生存时间等。相反，ISharedComponentData 适合很多实体共享相同数据的情况。例如在Boid demo中我们通过相同的（预制）Prefab实例化了大量的实体，因此在大量的Boid实体之间RenderMesh都是完全一样的。

```c#

[System.Serializable]
public struct RenderMesh : ISharedComponentData
{
    public Mesh                 mesh;
    public Material             material;

    public ShadowCastingMode    castShadows;
    public bool                 receiveShadows;
}

```

ISharedComponentData的最大优势是对于每一个实例的角度来看可以说是零内存消耗。

我们使用ISharedComponentData将所有InstanceRender数据相同的实体分组到一块，然后高效的提取所有的矩阵用于渲染。最终代码简洁而高效，这是因为数据完全是按照数据的访问方式来排布的。

+ RenderMeshSystemV2（参见代码文件Packages/com.unity.entities/Unity.Rendering.Hybrid/RenderMeshSystemV2.cs）

#### 关于共享组件数据的一些重要注意事项

+ 带有相同共享组件数据的实体在相同的内存块中分组到一起。共享组件数据的索引在每一个内存块中存储一份，而不是每个实体存一份。这样做的结果就是从每一个实体的角度来看共享组件数据都是零内存消耗的。
+ 使用实体查询（EntityQuery）我们可以遍历所有相同类型的实体。
+ 另外我们还可以使用EntityQuery。SetFilter（）来遍历带有指定共享组件数据的实体。由于数据排列方式这种遍历性能消耗很低。
+ 我们可以使用EntityMananger.GetAllUniqueSharedComponents来获取所有当前可用实体中添加的所有独一无二的共享组件数据。
+ 共享组件数据是自动引用计数的。
+ 共享组件数据应当很少变更。改变共享组件数据涉及到使用memcpy将实体中所有的组件数据拷贝到别的内存块中。

## 系统状态组件

系统状态组件存在的意义是允许你跟踪系统内部的资源，并且在不需要依赖单独的回调的前提下，可以合理的按需创建和销毁资源。

系统状态组件数据（SystemStateComponentData）和系统状态共享组件数据二者与组件数据和共享组件数据完全一致，严格的说，除了一个重要方面：系统状态组件数据在实体被销毁时不会被删除。

DestroyEntity 可以概括为以下三项内容：
+ 1.找到所有饮用特定实体ID的组件
+ 2.将找到的组件删除
+ 3.回收利用实体ID

然而如果存在系统状态组件数据的话，系统状态组件数据是不会被删除的。这使得系统可以清理任何与实体ID相关联的资源或者状态。只有当所有的系统状态组件数据都被删除的时候实体ID才会被回收。

#### 使用系统状态组件的动机

+ 系统可能需要根据组件数据维护内部状态。例如，某些资源可能需要被分配
+ 系统需要能够将状态按照值来管理，并且能够管理其他系统对状态值的修改。例如，当组件值的被修改时，或者相关组件被添加或删除时。
+ “不使用回调”是ECS设计规则中的重要元素

### 系统状态组件相关概念

通常来说系统状态组件数据都应该是某个用户组件的镜像，用于内部状态处理。

举例来说，假定有：

+ FooComponent（组件数据，用户添加）
+ FooStateComponent（系统组件数据，系统添加）

#### 检测组件添加

当用户添加FooComponent时，FooStateComponent 还不存在。FooSystem在更新对FooComponent的查询时候找不到FooStateComponent，并由此推断这个组件刚被添加进来。此时，FooSystem会添加FooStateComponent并同时添加所需的内部状态。

#### 检测组件删除

当用户删除FooComponent时，FooStateComponent还存在。FooSystem在更新对FooStateComponent的查询时找不到FooComponent，并由此推断组件被删除了。此时，FooSystem会删除FooStateComponent并修复任何需要的内部状态。

#### 检测销毁实体

正如上面所说的，DestroyEntity 实际上可以概括为以下三项内容：
+ 1.找到所有饮用特定实体ID的组件
+ 2.将找到的组件删除
+ 3.回收利用实体ID

然而，系统状态组件数据在DestroyEntity执行时不会被移除，实体ID直到最后一个组件被删除时才会被回收。这样做是的系统能够在组件被删除时使用相同的方法清理内部状态。

#### SystemStateComponent

SystemStateComponentData和ComponentData形式类似，用法也相似

```C#
struct FooStateComponent : ISystemStateComponentData
{
}
```

SystemStateComponetData的可访问性的控制方式也和组件一致（使用private，public，internal）然而有一条规则例外，即SystemStateComponentData在创建它的系统之外是只读的（ReadOnly）

#### SystemStateSharedComponent

SystemStateSharedComponent和SharedComponentData形式类似，用法也相似

```c#
struct FooStateSharedComponent : ISystemStateSharedComponentData
{
  public int Value;
}
```

#### 使用状态组件的系统示例

下面的示例掩饰了一个简单的系统如何使用系统状态组件来管理实体。示例定义了一个泛用的IComponentData实例和一个系统状态，即ISystemStateComponentData实例。示例还定一个了对实体的三组查询：

+ m_newEntities 筛选出带有泛用组件而不带系统状态组件的实体。这组查询查找到系统中刚刚出现的新实体。系统使用这个新实体的查询结果运行一个事务（job）来给这些实体添加系统状态组件。
+ m_activeEntities 筛选出既包含泛用组件又包含系统状态组件的实体。在真实的应用中，会使用其他系统来销毁实体。
+ m_destroyedEntities 筛选出有系统状态组件，但是没有泛用组件的实体。因为实体不能自己给自己添加系统状态组件，由这个查询筛选出的实体只能被删除，删除操作或是由该系统本身，或是由其他系统来执行。系统使用被销毁的的实体的查询结果来巡行一个事务（job）来从实体中删除系统状态组件，之后ECS代码就可以回收利用实体的标识符了。

注意这个简化的示例并不在系统内维护任何状态。系统状态组件的目的之一是记录需要分配或清理的常驻资源。

```c#

using Unity.Collections;
using Unity.Entities;
using Unity.Jobs;
using UnityEngine;

public struct GeneralPurposeComponentA : IComponentData
{
    public bool IsAlive;
}

public struct StateComponentB : ISystemStateComponentData
{
    public int State;
}

public class StatefulSystem : JobComponentSystem
{
    private EntityQuery m_newEntities;
    private EntityQuery m_activeEntities;
    private EntityQuery m_destroyedEntities;
    private EntityCommandBufferSystem m_ECBSource;

    protected override void OnCreate()
    {
        // Entities with GeneralPurposeComponentA but not StateComponentB
        m_newEntities = GetEntityQuery(new EntityQueryDesc()
        {
            All = new ComponentType[] {ComponentType.ReadOnly<GeneralPurposeComponentA>()},
            None = new ComponentType[] {ComponentType.ReadWrite<StateComponentB>()}
        });

        // Entities with both GeneralPurposeComponentA and StateComponentB
        m_activeEntities = GetEntityQuery(new EntityQueryDesc()
        {
            All = new ComponentType[]
            {
                ComponentType.ReadWrite<GeneralPurposeComponentA>(),
                ComponentType.ReadOnly<StateComponentB>()
            }
        });

        // Entities with StateComponentB but not GeneralPurposeComponentA
        m_destroyedEntities = GetEntityQuery(new EntityQueryDesc()
        {
            All = new ComponentType[] {ComponentType.ReadWrite<StateComponentB>()},
            None = new ComponentType[] {ComponentType.ReadOnly<GeneralPurposeComponentA>()}
        });

        m_ECBSource = World.GetOrCreateSystem<EndSimulationEntityCommandBufferSystem>();
    }

    struct NewEntityJob : IJobForEachWithEntity<GeneralPurposeComponentA>
    {
        public EntityCommandBuffer.Concurrent ConcurrentECB;

        public void Execute(Entity entity, int index, [ReadOnly] ref GeneralPurposeComponentA gpA)
        {
            // Add an ISystemStateComponentData instance
            ConcurrentECB.AddComponent<StateComponentB>(index, entity, new StateComponentB() {State = 1});
        }
    }

    struct ProcessEntityJob : IJobForEachWithEntity<GeneralPurposeComponentA>
    {
        public EntityCommandBuffer.Concurrent ConcurrentECB;

        public void Execute(Entity entity, int index, ref GeneralPurposeComponentA gpA)
        {
            // Process entity, possibly setting IsAlive false --
            // In which case, destroy the entity
            if (!gpA.IsAlive)
            {
                ConcurrentECB.DestroyEntity(index, entity);
            }
        }
    }

    struct CleanupEntityJob : IJobForEachWithEntity<StateComponentB>
    {
        public EntityCommandBuffer.Concurrent ConcurrentECB;

        public void Execute(Entity entity, int index, [ReadOnly] ref StateComponentB state)
        {
            // This system is responsible for removing any ISystemStateComponentData instances it adds
            // Otherwise, the entity is never truly destroyed.
            ConcurrentECB.RemoveComponent<StateComponentB>(index, entity);
        }
    }

    protected override JobHandle OnUpdate(JobHandle inputDependencies)
    {
        var newEntityJob = new NewEntityJob()
        {
            ConcurrentECB = m_ECBSource.CreateCommandBuffer().ToConcurrent()
        };
        var newJobHandle = newEntityJob.ScheduleSingle(m_newEntities, inputDependencies);
        m_ECBSource.AddJobHandleForProducer(newJobHandle);

        var processEntityJob = new ProcessEntityJob()
            {ConcurrentECB = m_ECBSource.CreateCommandBuffer().ToConcurrent()};
        var processJobHandle = processEntityJob.Schedule(m_activeEntities, newJobHandle);
        m_ECBSource.AddJobHandleForProducer(processJobHandle);

        var cleanupEntityJob = new CleanupEntityJob()
        {
            ConcurrentECB = m_ECBSource.CreateCommandBuffer().ToConcurrent()
        };
        var cleanupJobHandle = cleanupEntityJob.ScheduleSingle(m_destroyedEntities, processJobHandle);
        m_ECBSource.AddJobHandleForProducer(cleanupJobHandle);

        return cleanupJobHandle;
    }

    protected override void OnDestroy()
    {
        // Implement OnDestroy to cleanup any resources allocated by this system.
        // (This simplified example does not allocate any resources.)
    }
}


``` 

## 动态缓冲区

动态缓冲区（DynamicBuffer）是一种组件数据，这种组件数据提供了一种与实体相关联的容量可变，可伸缩的缓冲区。动态缓冲区看起来像是臃肿内部能够容纳一定数量元素的组件类型，但是能偶在内部容量耗尽时分配堆内存。

内存管理在使用动态缓冲区时时全自动的。与动态缓冲区相关的内存通过EntityManager来管理，因此当动态缓冲区组件被删除时，与之相关的对内存也被自动释放掉了。

动态缓冲区取代了固定数组，后者已被删除。

#### 声明缓冲区元素类型

要声明一个缓冲区，你需要使用将要放进缓冲区中的元素的类型来声明缓冲区。

```c#

// This describes the number of buffer elements that should be reserved
// in chunk data for each instance of a buffer. In this case, 8 integers
// will be reserved (32 bytes) along with the size of the buffer header
// (currently 16 bytes on 64-bit targets)
[InternalBufferCapacity(8)]
public struct MyBufferElement : IBufferElementData
{
    // These implicit conversions are optional, but can help reduce typing.
    public static implicit operator int(MyBufferElement e) { return e.Value; }
    public static implicit operator MyBufferElement(int e) { return new MyBufferElement { Value = e }; }

    // Actual value each buffer element will store.
    public int Value;
}

```

虽然声明元素的类型而不是缓冲区本身的类型有点奇怪，但是这种设计实现了ECS架构中的两个关键收益点：
+ 1. 这种设计不仅仅是支持一种float3型的动态缓存，或者任何其他常见的变量类型。只要元素分别单独包装到顶层结构体中，就可以添加任意数量的使用相同值类型的动态缓冲区。
+ 2.你可以将缓冲区元素类型包含到实体原型（EntityArchetypes）中，在外界看起来基本上就像是实体原型中的一个组件。

#### 向实体中添加缓冲区类型

想要向实体中添加一个缓冲区，你可以使用向实体添加组件类型的一般方法

##### 使用 AddBuffer()

```c#

entityManager.AddBuffer<MyBufferElement>(entity);

```

##### 使用原型

```c#

Entity e = entityManager.CreateEntity(typeof(MyBufferElement));

```

#### 访问缓冲区

在访问常规的组件数据的方法之外，有很多种访问动态缓冲区的方法

##### 直接的，仅在主线程上的访问

```c#

DynamicBuffer<MyBufferElement> buffer = entityManager.GetBuffer<MyBufferElement>(entity);

```

##### 基于实体访问

你也可以通过事物组件系统（JobComponentSystem）来逐个实体查找缓冲区

```c#

    var lookup = GetBufferFromEntity<MyBufferElement>();
    var buffer = lookup[myEntity];
    buffer.Append(17);
    buffer.RemoveAt(0);

```

#### 重新解析缓冲区（实验性）

缓冲区可以被重新解析为一个有相同大小的其他类型。想法是允许控制类型多义，并且摆脱当存在类型问题时对元素类型进行包装的操作。要使用重新解析，调用Reinterpret<T>即可

```c#

var intBuffer = entityManager.GetBuffer<MyBufferElement>().Reinterpret<int>();

```

重解析后的缓冲区带有原来缓冲区的安全句柄，可以安全使用。它们都在底层使用相同的缓冲区头（BufferHeader），因此对一个重解析后的缓冲区的修改会直接反应到其转化类型出缓冲区。

注意这里没有做类型检测，所以将unit缓冲区别名称为float缓冲区也是可以的。

#### 内存块组件数据

内存块组件用于将数据关联到特定的内存块。

内存块组件包含特定内存块中应用到所有实体上数据。例如，假设有一些代表3D对象的实体的内存块，这些实体由空间距离组织在一起，可以通过内存块组件在内存块中存储这些实体的包围盒。内存块组件使用接口类型IComponentData。

虽然内存块组件可以拥有对单个内存块来说唯一的值，它们仍然是内存块中实体原型的组成部分。因此如果将一个内存块组件从实体中移除，这个实体会被移动到其他的内存块中（有可能是一个新内存块）。类似的，如果向实体中添加一个内存块组件，以为实体的原型改变了，实体也会被移动到一个别的内存块里；增加内存块组件并不影响原来内存块中剩余的实体。

如果你改变了内存块中某个实体的内存块组件中的值，会同时改变所有共用内存块组件的实体的内存块组件的值。如果改变了某个组件的原型使得该实体被移动到一个新的内存块里，恰巧这个内存块里面有相同类型的内存块组件，那么目标内存块中的内存块组件的已有值会继续保留。（如果实体被移动到一个新建的内存块中，那么内存块组件的值会从实体的源内存块里被复制过来）

操作内存块组件和操作泛用组件的主要区别在于使用不同的方法来添加，设置和删除组件。内存块组件还有它们自己的ComponentType方法来定义实体原型和查询。

**相关的API**


|Purpose|Function|
|--- |--- |
|Declaration|IComponentData|
||ArchetypeChunk methods|
|Read|GetChunkComponentData(ArchetypeChunkComponentType)|
|Check|HasChunkComponent(ArchetypeChunkComponentType)|
|Write|SetChunkComponentData(ArchetypeChunkComponentType, T)|
||EntityManager methods|
|Create|AddChunkComponentData(Entity)|
|Create|AddChunkComponentData(EntityQuery, T)|
|Create|AddComponents(Entity,ComponentTypes)|
|Get type info|GetArchetypeChunkComponentType(Boolean)|
|Read|GetChunkComponentData(ArchetypeChunk)|
|Read|GetChunkComponentData(Entity)|
|Check|HasChunkComponent(Entity)|
|Delete|RemoveChunkComponent(Entity)|
|Delete|RemoveChunkComponentData(EntityQuery)|
|Write|EntityManager.SetChunkComponentData(ArchetypeChunk, T)|

##### 声明一个内存块组件

内存块组件使用接口类型IComponentData

```c#
public struct ChunkComponentA : IComponentData
{
    public float Value;
}
```

##### 创建一个内存块组件

可以通过使用目标内存块中的实体或通过实体查询选择一组目标内存块来直接添加内存块组件。内存块组件不能在事务中添加，也不能通过EntityCommandBuffer来添加。

你还可以将内存块组件当作EntityArchetype或用于创建实体的ComponentType列表的一部分，此时内存块组件会为每个存储该原型的实体的内存块中创建。在这些方法中使用ComponentType.ChunkComponent<T>或者ComponentType.ChunkComponentReadOnly<T>,否则组件会被当作泛用组件处理。

+ 使用内存块中的某个实体创建内存块组件

对于目标内存块中的某个实体，可以使用EntityMananger.AddChunkComponentData<T>方法来向内存块汇总添加内存块组件：

```C#
EntityManager.AddChunkComponentData<ChunkComponentA>(entity);
```

使用该方法，不能立即对实体组件设置值

+ 使用实体查询(EntityQuery)创建内存块组件

对选中所有需要添加内存块组件的实体查询，可以添加组件并设置组件值，这一操作使用EntityManager.AddChunkComponentData<T>()方法：

```C#
EntityQueryDesc ChunksWithoutComponentADesc = new EntityQueryDesc()
{
    None = new ComponentType[] {ComponentType.ChunkComponent<ChunkComponentA>()}
};
ChunksWithoutChunkComponentA = GetEntityQuery(ChunksWithoutComponentADesc);

EntityManager.AddChunkComponentData<ChunkComponentA>(ChunksWithoutChunkComponentA,
        new ChunkComponentA() {Value = 4});
```

使用该方法，可以对所有新内存块组件设置相同的初始值

+ 使用实体原型(EntityArchetype)

当使用原型或组件类型列表创建实体时，将内存块组件包含在原型中：

```c#
ArchetypeWithChunkComponent = EntityManager.CreateArchetype(
    ComponentType.ChunkComponent(typeof(ChunkComponentA)),
    ComponentType.ReadWrite<GeneralPurposeComponentA>());
var entity = EntityManager.CreateEntity(ArchetypeWithChunkComponent);
```

或者使用组件类型列表

```C#
ComponentType[] compTypes = {ComponentType.ChunkComponent<ChunkComponentA>(),
                             ComponentType.ReadOnly<GeneralPurposeComponentA>()};
var entity = EntityManager.CreateEntity(compTypes)
```

通过上述方法，新内存块的内存块组件在实体的构造过程中创建，初始化为结构的默认值。已存在的内存块中的内存块组件不变。参照《更新内存块组件》这一节来了解如何对给定的实体引用设置内存块组件。

##### 读取内存块组件

可以通过目标内存块中的某个实体来读取内存块组件，也可以使用代表内存块的ArchetypeChunk对象来读取。

+ 通过内存块中的实体读取内存块组件

对于某个实体，可以使用EntityManager。GetChunkComponentData<T>来访问某个内存块组件：

```c#
if(EntityManager.HasChunkComponent<ChunkComponentA>(entity))
    chunkComponentValue = EntityManager.GetChunkComponentData<ChunkComponentA>(entity);
```

也可以建立一个fluent query来选择金包含某个内存块组件的实体：

```c#
Entities.WithAll(ComponentType.ChunkComponent<ChunkComponentA>()).ForEach(
    (Entity entity) =>
{
    var compValue = EntityManager.GetChunkComponentData<ChunkComponentA>(entity);
    //...
});
```

注意不能在查询的for-each循环中传递内存块组件。相反，必须传递实体对象，并使用EntityManager来访问内存块组件。

+ 通过ArchetypeChunk实例来读取内存块组件

对于某个内存块，可以使用EntityManager.GetChunkComponentData<T>方法来读取内存块组件。下列代码遍历所有内存块找到符合的查询，然后访问ChunkComponentA类型的内存块组件：

```c#
var chunks = ChunksWithChunkComponentA.CreateArchetypeChunkArray(Allocator.TempJob);
foreach (var chunk in chunks)
{
    var compValue = EntityManager.GetChunkComponentData<ChunkComponentA>(chunk);
    //..
}
chunks.Dispose();
```
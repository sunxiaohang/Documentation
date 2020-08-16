## UnityEditor

### 扩展Project视图
#### 扩展右键菜单
```C#
//[MenuItem("Assets/Tools/Debug",isValidateFunction,priority)]
public class Extension
{
    [MenuItem("Assets/CustomTools/Debug",false,2)]
    static void DebugFunction()
    {
        Debug.Log(Selection.activeObject.name);
    }
}
```
#### 创建菜单
```C#
//[MenuItem("Assets/Create/ObjectName",isValidateFunction,priority)]
public class Extension
{
    [MenuItem("Assets/Create/Debug",false,2)]
    static void CreateObjectName()
    {
        GameObject.CreatePrimitive(PrimitiveType.Sphere);
    }
}
```
#### 扩展布局
```C#
public class Extension
{
    //表示此方法会在C#代码每次编译完成后首先调用
    [InitializeOnLoadMethod]
    static void Function()
    {
        //监听EditorApplication.projectWindowItemOnGUI委托，即可使用GUI方法来绘制自定义UI元素
        EditorApplication.projectWindowItemOnGUI = delegate(string guid, Rect rect) { 
            //在project视图中选择一个资源
            if (Selection.activeObject &&guid == AssetDatabase.AssetPathToGUID(AssetDatabase.GetAssetPath(Selection.activeObject)))
            {
                //设置拓展按钮区域
                float width = 50f;
                rect.x += rect.width - width;
                rect.y += 2f;
                rect.width = width;
                GUI.color = Color.red;
                //点击事件处理的简单写法
                if(GUI.Button(rect,"click")){
                    Debug.LogFormat("click:{0}",Selection.activeObject.name); 
                };
                GUI.color= Color.white;
            }
        };
    }
}
```

#### 监听事件
```C#
//事件监听需要继承父类UnityEditor.AssetModificationProcessor
public class Extension:UnityEditor.AssetModificationProcessor{
    //全局监听Project视图下的资源是否发生变化
    [InitializeOnLoadMethod]
    static void Function(){
        EditorApplication.projectChanged += delegate(){Debug.Log("change");};
    }
    //监听双击鼠标左键，打开资源事件
    public static bool IsOpenForEdit(string assetPath, out string message){
        return true;//true 表示该资源可以打开，false表示不允许在unity中打开该资源
    }
    //监听资源即将被创建
    private static void OnWillCreateAsset(string assetName){ 
        Debug.LogFormat("path:{0}",assetName);
    }
    //监听资源即将被保存
    private static string[] OnWillSaveAssets(string[] paths){
        if (paths != null)Debug.LogFormat("path:{0}",string.Join(",",paths));
        return paths;
    }
    //监听资源即将被移动
    private static AssetMoveResult OnWillMoveAsset(string sourcePath, string destinationPath){
        Debug.LogFormat("from:{0} to {1}",sourcePath,destinationPath);
        return AssetMoveResult.DidMove;//AssetMoveResult.DidMove表示资源可以移动
    }
    //监听资源即将被删除
    private static AssetDeleteResult OnWillDeleteAsset(string assetPath, RemoveAssetOptions options){
        Debug.LogFormat("delete:{0}",assetPath);
        return AssetDeleteResult.DidNotDelete;//表示该资源可以被删除
    }
}
```
### 拓展Hierarchy视图
#### 拓展菜单
```C#
public class Extension{
    [MenuItem("GameObject/CustomCreate/Cube", false, 1)]
    static void CreateCube()
    {
        GameObject.CreatePrimitive(PrimitiveType.Cube);
    }
}
```
#### 拓展布局
```C#
public class Extension
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoad()
    {
        EditorApplication.hierarchyWindowItemOnGUI = delegate(int id, Rect rect) {  
            //在hierarchy视图中选择一个资源
            if (Selection.activeObject && id == Selection.activeObject.GetInstanceID())
            {
                float width = 50f;
                float height = 20f;
                rect.x = rect.width - width;
                rect.width = width;
                rect.height = height;
                if (GUI.Button(rect, AssetDatabase.LoadAssetAtPath<Texture>("Assets/Unity.png")))
                {
                    Debug.LogFormat("click:{0}",Selection.activeObject.name);
                }
            }
        };
    }
}
```
#### 重写菜单
```C#
public class Extension
{
    [InitializeOnLoadMethod]
    static void StartInitializeOnLoadMethod() {
        EditorApplication.hierarchyWindowItemOnGUI += OnHierarchyGUI;
    }
    static void OnHierarchyGUI(int instanceId, Rect selectionRect) {
        if (Event.current != null 
            && selectionRect.Contains(Event.current.mousePosition)
            &&Event.current.button==1&&Event.current.type<=EventType.MouseUp) {
            GameObject selectedGameObject = EditorUtility.InstanceIDToObject(instanceId) as GameObject;
            if (selectedGameObject) {
                Vector2 mousePosition = Event.current.mousePosition;
                EditorUtility.DisplayPopupMenu(new Rect(mousePosition.x,mousePosition.y,0,0),"Window/Test",null );
                Event.current.Use();
            }
        }
    }
    [MenuItem("Window/Test/tools1")]
    static void Test() {}
    [MenuItem("Window/Test/tools2")]
    static void Test1() { }
    [MenuItem("Window/Test/tools3")]
    static void Test2() { }
}
```
### 拓展Inspector视图
#### 拓展原生组件
```C#
[CustomEditor(typeof(Camera))]
public class Extension:Editor{
    public override void OnInspectorGUI(){
        if (GUILayout.Button("extensionButton")) {/*function*/ }
        base.OnInspectorGUI();
    }
}
```
#### 拓展继承组件
Unity将大量的Editor绘制方法封装在内部DLL文件里，开发者无法调用它的方法，只能通过反射的方式调用内部未公开的方法。
```C#
[CustomEditor(typeof(Transform))]
public class Extension:Editor{
    private Editor m_Editor;
    private void OnEnable(){
        m_Editor = Editor.CreateEditor(target,     Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.TransformInspector", true));
    }
    public override void OnInspectorGUI(){
        if (GUILayout.Button("extensionButton")) { }
        GUI.enabled = false;//------------开始禁止
        m_Editor.OnInspectorGUI();//调用系统绘制方法
        GUI.enabled = true;//------------结束禁止
        if (GUILayout.Button("extensionButton2")) { }
    }
}
```
#### 组件不可编辑
```C#
HideFlag可以使用按位或(|)同时保持多个属性
GUI.enabled = false;//------------开始禁止
////m_Editor.OnInspectorGUI();//调用系统绘制方法
GUI.enabled = true;//------------结束禁止
//全局锁定与解锁修改
public class Extension{
    [MenuItem("GameObject/3D Object/Lock/Lock")]
    static void Lock(){
        if (Selection.gameObjects != null){
            foreach (var gameObject in Selection.gameObjects)                                          gameObject.hideFlags = HideFlags.NotEditable;
        }
    }
    [MenuItem("GameObject/3D Object/Lock/Unlock")]
    static void Unlock(){
        if (Selection.gameObjects != null){
            foreach (var gameObject in Selection.gameObjects) {
                gameObject.hideFlags = HideFlags.None;
            }
        }
    }
}
```
#### Context菜单
```C#
Inspector中的组件右键菜单，如果想给所有组件都添加上该菜单Transform>>>Component
public class Extension {
    [MenuItem("CONTEXT/Transform/NewContext1")]
    static void NewContext1(MenuCommand command) {
        Debug.Log(command.context.name);
    }
}
该设置也可以用在自己写的脚本中,用于读取和写入组件属性
using UnityEngine;
#if UNITY_EDITOR
using UnityEditor;
#endif
public class Extension:MonoBehaviour{
    public string contextName;
    #if UNITY_EDITOR
    [MenuItem("CONTEXT/Extension/NewContext1")]
    static void NewContext1(MenuCommand command) {
        Extension extension = command.context as Extension;
        if(extension!=null)extension.contextName = "HelloWorld";
    }
    #endif
}
//#if UNITY_EDITOR     #endif只会在Editor模式下执行，发布后会被剔除
如果自定义菜单和系统菜单项目重名，还可以覆盖
[ContextMenu("Remove Component")]
void RemoveComponent(){
    Debug.Log("RemoveComponent");
    //等一会在删除自己
    UnityEditor.EditorApplication.delayCall = delegate(){DestoryImmediate(this);};
}
```
### 拓展Scene视图
#### 辅助元素（该类不在Editor目录下）
```C#
public class Extension:MonoBehaviour{
    private void OnDrawGizmosSelected(){
        Gizmos.color = Color.red;
        Gizmos.DrawLine(transform.position,Vector3.zero);
        Gizmos.DrawCube(Vector3.zero, Vector3.one);
        //其他Gizmos.function()
    }
    //辅助元素不依赖选择对象出现(始终显示zaiScene视图中)
    private void OnDrawGizmos(){
        Gizmos.DrawSphere(transform.position,1);
    }
}
```
#### 辅助UI
在Scene中我们可以添加EditorGUI，方便在视图中处理一些操作。EditorGUI的代码需要在Handles.BeginGUI()和Handles.EndGUI()中间绘制完成。
```C#
[CustomEditor(typeof(Camera))]
public class Extension:Editor{
    private void OnSceneGUI(){
        Camera camera = target as Camera;
        if (camera != null){
            Handles.color = Color.red;
         Handles.Label(camera.transform.position,camera.transform.position.ToString());
            Handles.BeginGUI();
            GUI.backgroundColor = Color.red;
            if (GUILayout.Button("click", GUILayout.Width(200f))){
                Debug.LogFormat("click={0}",camera.name);
            }
            GUILayout.Label("Label");
            Handles.EndGUI();
        }
    }
}
```
#### 常驻辅助UI
```C#
public class Extension{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod(){
        SceneView.duringSceneGui += delegate(SceneView sceneView){
            Handles.BeginGUI();
            GUI.Label(new Rect(0,0,50,15),"title" );
            GUI.Button(new Rect(0, 20, 50, 50)
                , AssetDatabase.GetBuiltinExtraResource<Texture>("Assets/unity.png"));
            Handles.EndGUI();
        };
    }
}
```
#### 禁用选中对象
```C#
public class Extension{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod(){
        SceneView.duringSceneGui += delegate(SceneView sceneView){
            Event e = Event.current;
            if (e != null){
                //获取它的controllID后，即可禁止将点击事件穿透下去
                //表示禁止接收控制焦点
                int controlID = GUIUtility.GetControlID(FocusType.Passive);
                if (e.type == EventType.Layout){
                    HandleUtility.AddDefaultControl(controlID);
                }
            }
        };
    }
}
```
### 拓展Game视图
Game视图拓展分1.运行模式2.非运行模式
```C#
#if UNITY_EDITOR
[ExecuteInEditMode]//非运行模式下也会执行代码的生命周期
public class Extension:MonoBehaviour{
    void OnGUI(){
        if(GUILayout.Button("Click")){
            Debug.Log("Click!");
        }
        GUILayout.Label("HelloWorld");
    }
}
#endif
```
### MenuItem菜单
#### 覆盖系统菜单
```C#
[MenuItem("GameObject/UI/Text")]
static void CreateNewText(){
    //todo
}
自定义菜单
[MenuItem("Root/TestCheck",false,1)]
static void checkFunction() {
    var menuPath = "Root/TestCheck";
    bool m_Checked = Menu.GetChecked(menuPath);
    Menu.SetChecked(menuPath,!m_Checked);
}
[MenuItem("Root/TestGray")]
static void grayFunction() { }
[MenuItem("Root/TestGray",true,20)]
static bool grayFunctionValidate() {
    return false;//false表示置灰，不可点击
}
```
### 面板拓展
#### Inspector面板
```C#
#if UNITY_EDITOR
using UnityEditor;
#endif
using UnityEditor.Experimental.TerrainAPI;
using UnityEngine;
#if UNITY_EDITOR
[CustomEditor(typeof(PoJo))]
public class Extension:Editor
{
    private bool m_EnableToogle;
    public override void OnInspectorGUI()
    {
        PoJo poJo = target as PoJo;
        poJo.scrollPos = EditorGUILayout.BeginScrollView(poJo.scrollPos, false, true);
        poJo.myName = EditorGUILayout.TextField("text", poJo.myName);
        poJo.myId = EditorGUILayout.IntField("int", poJo.myId);
        poJo.prefab = EditorGUILayout.ObjectField("GameObject", poJo.prefab,typeof(GameObject),true)as GameObject;
        //绘制按钮
        EditorGUILayout.BeginHorizontal();
        GUILayout.Button("1");
        GUILayout.Button("2");
        poJo.myEnum = (PoJo.MyEnum) EditorGUILayout.EnumPopup("MyEnum:", poJo.myEnum);
        EditorGUILayout.EndHorizontal();
        
        //toogle component
        m_EnableToogle = EditorGUILayout.BeginToggleGroup("EnableToogle", m_EnableToogle);
        poJo.toogle1 = EditorGUILayout.Toggle("toogle1", poJo.toogle1);
        poJo.toogle2 = EditorGUILayout.Toggle("toogle2", poJo.toogle2);
        EditorGUILayout.EndToggleGroup();
        EditorGUILayout.EndScrollView();
    }
}
#endif
```
### 编辑器窗口Editor Window
```C#
public class CustomWindow:EditorWindow,IHasCustomMenu
{
    [MenuItem("Window/CustomWindow")]
    static void Init(){
        CustomWindow window = (CustomWindow) EditorWindow.GetWindow(typeof(CustomWindow));
        window.Show();
    }
    private Texture m_MyTexture = null;
    private float m_MyFloat = 0.5f;
    private void Awake(){
        Debug.LogFormat("窗口初始化调用");
        m_MyTexture = AssetDatabase.LoadAssetAtPath<Texture>("Assets/unity.png");
    }
    private void OnGUI(){
        GUILayout.Label("HelloWorld",EditorStyles.boldLabel);
        m_MyFloat = EditorGUILayout.Slider("Slide", m_MyFloat, -5, 5);
        GUI.DrawTexture(new Rect(0,30,100,100),m_MyTexture);
    }
    private void OnDestroy(){/*销毁时调用*/}
    private void OnFocus(){/*拥有焦点时调用*/}
    private void OnHierarchyChange(){/*hierarchy视图发生变化时调用*/}
    private void OnInspectorUpdate(){/*Inspector每帧更新*/}
    private void OnLostFocus(){ /*失去焦点时调用*/}
    private void OnProjectChange(){/*Project视图发生变化时调用*/}
    private void OnSelectionChange(){/*在hierarchy或project视图中选择一个对象时调用*/}
    private void Update(){/*每帧调用*/}
    public void AddItemsToMenu(GenericMenu menu){
        menu.AddDisabledItem(new GUIContent("Disable"));
        menu.AddItem(new GUIContent("Test1"),true, () =>{/*todo*/});
        menu.AddSeparator("Test/");
        menu.AddItem(new GUIContent("Test/Test3"),true, () => { });
    }
}
```
### 自定义导入类型
```C#
[ScriptedImporter(1,"custom")]
public class ImportListener : ScriptedImporter{
    public override void OnImportAsset(AssetImportContext ctx){
        var cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
        var position = JsonUtility.FromJson<Vector3>(File.ReadAllText(ctx.assetPath));
        cube.transform.position = position;
        cube.transform.localScale = Vector3.one;
        ctx.AddObjectToAsset("obj",cube);
        ctx.SetMainObject(cube);
        //添加材质
        var material = new Material(Shader.Find("Standard"));
        material.color = Color.red;
        ctx.AddObjectToAsset("material",material);
        var tempMesh = new Mesh();
        DestroyImmediate(tempMesh);
    }
}
```
### 游戏脚本
#### 脚本序列化
```C#
#if UNITY_EDITOR
[CustomEditor(typeof(SerializedClass))]
public class ScriptInspector : Editor{
    public override void OnInspectorGUI(){
        serializedObject.Update();//更新最新数据
        EditorGUI.BeginChangeCheck();//标记检查
        SerializedProperty property = serializedObject.FindProperty("id");//获取数据信息
        property.intValue = EditorGUILayout.IntField("主键", property.intValue);//保存数据
        property = serializedObject.FindProperty("name");
        property.stringValue = EditorGUILayout.TextField("姓名", property.stringValue);
        property = serializedObject.FindProperty("prefab");
        property.objectReferenceValue = EditorGUILayout.ObjectField("游戏对象",property.objectReferenceValue,typeof(GameObject),true);
        EditorGUILayout.PropertyField(serializedObject.FindProperty("targets"), true);
        //标记检查发生变化
        if (EditorGUI.EndChangeCheck());//标记检查发生变化
        if (GUI.changed);//判断面板元素变化
        //保存全部数据
        serializedObject.ApplyModifiedProperties();
    }
}
#endif
```
#### 序列化反序列化
```C#
public class SerializedClass:MonoBehaviour,ISerializationCallbackReceiver{
    [SerializeField] private List<Sprite> m_Values = new List<Sprite>();
    [SerializeField]private List<string> m_Keys = new List<string>();
    public Dictionary<string, Sprite> m_SpriteDic = new Dictionary<string, Sprite>();
    public void OnBeforeSerialize(){
        m_Keys.Clear();
        m_Values.Clear();
        foreach (KeyValuePair<string,Sprite> pair in m_SpriteDic){
            m_Keys.Add(pair.Key);
            m_Values.Add(pair.Value);
        }
    }
    public void OnAfterDeserialize(){
        m_SpriteDic.Clear();
        for (int i = 0; i < m_Keys.Count; i++){
            m_SpriteDic[m_Keys[i]] = m_Values[i];
        }
    }
}
#if UNITY_EDITOR
[CustomEditor(typeof(SerializedClass))]
public class Extension:Editor{
    public override void OnInspectorGUI(){
        serializedObject.Update();
        SerializedProperty propertyKey = serializedObject.FindProperty("m_Keys");
        SerializedProperty propertyValue = serializedObject.FindProperty("m_Values");
        GUILayout.BeginVertical();
        for (int i = 0; i < propertyKey.arraySize; i++){
            GUILayout.BeginHorizontal();
            SerializedProperty key = propertyKey.GetArrayElementAtIndex(i);
            SerializedProperty value = propertyValue.GetArrayElementAtIndex(i);
            key.stringValue = EditorGUILayout.TextField("Key", key.stringValue);
            value.objectReferenceValue =
                EditorGUILayout.ObjectField("Value", value.objectReferenceValue, typeof(Sprite),false);
            GUILayout.EndHorizontal();
        }
        GUILayout.EndVertical();
        GUILayout.BeginHorizontal();
        if (GUILayout.Button("+")){
            (target as SerializedClass).m_SpriteDic[propertyKey.arraySize.ToString()] = null;
        }
        GUILayout.EndHorizontal();
        serializedObject.ApplyModifiedProperties();
    }
}
#endif
```
### Scriptable Object
```C#
//菜单点击生成
[CreateAssetMenu]
public class Displayer : ScriptableObject{
    [SerializeField] public List<DisplayerInfo> m_DisplayerInfo;
    [Serializable]public class DisplayerInfo{
        public int id;
        public string name;
    }
}
public class DisplayerReader : MonoBehaviour{
    private void Start(){
        Displayer display = Resources.Load<Displayer>("Path");//读取
    }
}
public class Extension{
    [MenuItem("Assets/Create ScriptableObject")]
    static void CreateScriptableObject(){
        Displayer displayer = ScriptableObject.CreateInstance<Displayer>();
        displayer.m_DisplayerInfo = new List<Displayer.DisplayerInfo>();
        displayer.m_DisplayerInfo.Add(new Displayer.DisplayerInfo() {id = 100, name = "test"});
        //将资源保存到本地
   AssetDatabase.CreateAsset(displayer,"Assets/Resources/displayer.asset");
        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();
    }
}
```
### 单例脚本
切换场景对象销毁，挂载在对象上的脚本也被销毁，虽然Unity提供了DontDestoryOnLoad()方法，可以在切换场景是时不卸载游戏对象。PoJo脚本在他自己的static构造方法中构建对象并且static方法中设置DontDestoryOnLoad(),这样就能保证他自己不被主动卸载掉，并且构造方法只会执行一次。有些功能比较单一且需要用到脚本生命周期方法的类，就比较适合使用单例脚本，单例脚本的特点是它必须依赖游戏对象，并且必须保证这个游戏对象不能被卸载掉。Global脚本不需要在编辑器模式下绑定在某个对象上，运行是直接获取它的实例Global.instance.DoSomeThings();
```C#
public class Global : MonoBehaviour {
    public static Global instance;
    static Global() {
        GameObject go = new GameObject("#Global#");
        DontDestroyOnLoad(go);
        instance = go.AddComponent<Global>();
    }
    public void DoSomeThings();
}
```
### 定时器
协程任务是可以做定时器的，但是有个最大的问题，就是必须用在脚本中，逻辑大部分都在C#代码中，所以需要封装一个不依赖于脚本实现的定时器。利用协程程序来做定时器，给WaitTime()传入定时时间和结束后的回调方法，外部代码就可以处理定时解决的事件了。
```C#
public class WaitTimeManager{
    class TaskBehaviour : MonoBehaviour {}
    private static TaskBehaviour m_Task;
    static WaitTimeManager() {
        GameObject go = new GameObject("#WaitTimeManager#");
        GameObject.DontDestroyOnLoad(go);
        m_Task = go.AddComponent<TaskBehaviour>();
    }
    static public Coroutine WaitTime(float time, UnityAction callback){
        return m_Task.StartCoroutine(Coroutine(time,callback));
    }
    static public void CancelWait(ref Coroutine coroutine){
        if (coroutine != null){
            m_Task.StopCoroutine(coroutine);
            coroutine = null;
        }
    }
    static IEnumerator Coroutine(float time, UnityAction callback){
        yield return new WaitForSeconds(time);
        if (null!=callback) callback();
    }
}
//usage
Coroutine coroutine = WaitTimeManager.WaitTime(5f, delegate{});
WaitTimeManager.CancelWait(ref coroutine);
public class CustomYieldInstruction : MonoBehaviour{
    IEnumerator Start(){
        yield return new CustomWait(10f, 1f, delegate(){
            Debug.LogFormat("每过一秒回调一次:{0}",Time.time);
        });
    }
    public class CustomWait:CustomYieldInstruction{
        private UnityAction m_IntervalCallback;
        private float m_StartTime;
        private float m_LastTime;
        private float m_Interval;
        private float m_Time;
        public bool keepWaiting{
            get{
                if (Time.time - m_StartTime >= m_Time){
                    return false;
                }else if (Time.time - m_LastTime >= m_Interval){
                    m_LastTime = Time.time;
                    m_IntervalCallback();
                }
                return true;
            }
        }
        public CustomWait(float time, float interval, UnityAction callback){
            m_StartTime = Time.time;
            m_LastTime = Time.time;
            m_Interval = interval;
            m_Time = time;
            m_IntervalCallback = callback;
        }
    }        
}
```
### 工作线程
```C#
public class SerializedClass:MonoBehaviour{
    public Transform[] cubes;
    private void Update(){
        if (Input.GetMouseButtonDown(0)){
            NativeArray<Vector3> position = new NativeArray<Vector3>(cubes.Length,Allocator.Persistent);
            for (int i = 0; i < position.Length; i++){
                position[i] = Vector3.one*i;
            }
            TransformAccessArray transformAccessArray = new TransformAccessArray(cubes);
            ExecuteJob job = new ExecuteJob(){position = position};
            JobHandle jobHandle = job.Schedule(transformAccessArray);
            jobHandle.Complete();
            //工作线程结束
            transformAccessArray.Dispose();
            position.Dispose();
        }
    }
    struct ExecuteJob:IJobParallelForTransform{
        [ReadOnly] public NativeArray<Vector3> position;
        public void Execute(int index, TransformAccess transform){
            transform.position = position[index];
        }
    }
}
```
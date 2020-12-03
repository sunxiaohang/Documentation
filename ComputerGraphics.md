### GLSL关键定义
EBO(Element Buffer Object)索引缓冲对象：存储顶点的索引信息
VBO(Vertex Buffer Object)顶点缓冲对象：存储顶点的信息CPU(copy到)=>GPU
VAO(Vertex Array Object)顶点数组对象:存储了所有顶点数据属性的状态组合(顶点数据格式以及顶点数据所需的VBO对象引用)，相当于对多个VBO的引用
### GLSL一般流程
- init GLSL
- VertexShader
- FragmentShader
- LinkShader
- InitShapeData
- VBO VAO EBO
- Every frame update
- Clear data and storage usage
- Terminal
### OPENGL常用函数汇总
- void glfwInit() 初始化
- void glfwWindowHint(hint,value) 设置相关参数为期望值
- GLFWwindow* glfwCreateWindow(width,height,"name",NULL,NULL) 创建窗体
- glViewport(0,0,width,height) 设定视口大小
- glfwMakeContextCurrent(window) 设定上下文为指定窗口
- glfwSetFramebufferSizeCallback(window,framebuffer_size_callback) 窗口大小改变回调
- gladLoadGLLoader((GLADloadproc)glfwGetProcAddress) 加载GLAD
- glfwWindowShouldClose(window) 检查窗口是否关闭
- glfwTerminate() 结束程序
- glfwGetKey(window,GLFW_KEY_XXXX) 获取键盘输入
  
#### Shader相关
- int glCreateShader(GL_VERTEX_SHADER/GL_FRAGMENT_SHADER) 创建顶点片元Shader
- glShaderSource(vertexshader,1,"sourcecode",NULL) 绑定shader和源码
- glCompileShader(vertexShader) 编译shader源码
  
- int glCreateProgram() 链接shader
- glAttachShader(shaderProgram, vertexShader); attach顶点着色器
- glAttachShader(shaderProgram, fragmentShader);attach片元着色器
- glLinkProgram(shaderProgram); 链接程序

#### VBO VAO EBO
- glGenVertexArrays(1, &VAO);
- glGenBuffers(1, &VBO);
- glGenBuffers(1, &EBO);
- glBindVertexArray(VAO);
- glBindBuffer(GL_ARRAY_BUFFER, VBO);
- glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

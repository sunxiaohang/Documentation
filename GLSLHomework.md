### GLSL homework
##### create window with auto size change callback and key input
```
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

const unsigned int SRC_WIDTH = 800;
const unsigned int SRC_HEIGHT = 600;

void WindowSizeChangeCallback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow* window);
int main()
{
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR,3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR,3);
	glfwWindowHint(GLFW_OPENGL_PROFILE,GLFW_OPENGL_CORE_PROFILE);
	GLFWwindow* mainWindow = glfwCreateWindow(SRC_WIDTH,SRC_HEIGHT,"GLSL Tutorial",NULL,NULL);
	
	if (mainWindow == NULL)
	{
		std::cout << "create window failed" << std::endl;
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(mainWindow);
	glfwSetWindowSizeCallback(mainWindow,WindowSizeChangeCallback);
	//加载指针管理类
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		std::cout <<"GLAD load error" <<std::endl;
		glfwTerminate();
		return -1;
	}
	glViewport(0,0,SRC_WIDTH,SRC_HEIGHT);
	while (!glfwWindowShouldClose(mainWindow))
	{
		processInput(mainWindow);
		glClearColor(1.0f,0.2f,0.2f,1.0f);//RGBA
		glClear(GL_COLOR_BUFFER_BIT);//可选的参数为GL_COLOR_BUFFER_BIT,GL_DEPTH_BUFFER_BIT,GL_STENCIL_BUFFER_BIT分别标识颜色，深度，模板
		glfwSwapBuffers(mainWindow);//该函数会交换颜色缓冲（它是村塾GLFW窗口每一个像素颜色值的最大缓冲）
		glfwPollEvents();//检查有没有出发什么事件，比如键盘输入，鼠标移动等，更新窗口状态，并调用对应的回调函数
	}
	glfwTerminate();
	return 0;
}

void WindowSizeChangeCallback(GLFWwindow* window, int width, int height)
{
	glViewport(0, 0, width, height);
}
///--------------处理输入事件
void processInput(GLFWwindow* window)
{
	if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
	{
		glfwSetWindowShouldClose(window,true);
	}
}
```
##### Draw single triangle
- Vertex Array Object VAO
- Vertex Buffer Object VBO
- Element Buffer Object EBO|IBO
在openGL中任何事物都在3D空间中，而屏幕和窗口确实2D像素数组，这导致OpenGL大部分工作都是3D>>2D,该过程由OpenGL图形渲染管线管理，主要分为两个部分
- 3D坐标转换为2D坐标
- 2D坐标转换为实际的有颜色的像素(2D坐标是精确值，2D像素是近似值)
为了让OpenGL知道坐标和颜色构成，OpenGL需要指定这些数据的渲染类型
- GL_POINTS
- GL_TRIANGLES
- GL_LINE_STRIP
图形渲染管线几个部分
- 顶点着色器：把一个单独的顶点作为输入，主要目的3D坐标的坐标系转换(以及顶点属性转换)
- 图元装配：将顶点着色器的输出作为输入，区分顶点、线段、三角形
- 几何着色器：图元装配输出作为输入，通过产生新顶点构造出新图元来生成其他图形
- 光栅化阶段：把图元映射为最终屏幕上的像素点，生成供片段着色器使用的片段，在片段着色器运行前会进行裁切(超出视图范围的像素)
- 片段着色器：计算一个像素的最终颜色，也是OpenGL高级效果产生的地方(光照，阴影，光的颜色等)
- Alpha测试和混合：深度和Stencil值测试，决定是否丢弃，也会检测alpha值进行颜色混合

```
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

const unsigned int SRC_WIDTH = 800;
const unsigned int SRC_HEIGHT = 600;
const float vertices[] = {
	-0.5f,-0.5f,0.0f,
	0.5f,-0.5f,0.0f,
	0.0f,0.5f,0.0f
};
const char *vertexShaderSource = "#version 330 core\n"
"layout (location = 0) in vec3 aPos;\n"
"void main()\n"
"{\n"
"   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
"}\0";
const char *fragmentShaderSource = "#version 330 core\n"
"out vec4 FragColor;\n"
"void main()\n"
"{\n"
"   FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
"}\n\0";
void WindowSizeChangeCallback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow* window);
void processVerticeslist();
void processShader();
int main()
{
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR,3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR,3);
	glfwWindowHint(GLFW_OPENGL_PROFILE,GLFW_OPENGL_CORE_PROFILE);
	GLFWwindow* mainWindow = glfwCreateWindow(SRC_WIDTH,SRC_HEIGHT,"GLSL Tutorial",NULL,NULL);
	
	if (mainWindow == NULL)
	{
		std::cout << "create window failed" << std::endl;
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(mainWindow);
	glfwSetWindowSizeCallback(mainWindow,WindowSizeChangeCallback);
	//加载指针管理类
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		std::cout <<"GLAD load error" <<std::endl;
		glfwTerminate();
		return -1;
	}
	glViewport(0,0,SRC_WIDTH,SRC_HEIGHT);
	processVerticeslist();//处理顶点信息
	processShader();//处理渲染shader
	while (!glfwWindowShouldClose(mainWindow))
	{
		processInput(mainWindow);
		glClearColor(1.0f,0.2f,0.2f,1.0f);//RGBA
		glClear(GL_COLOR_BUFFER_BIT);//可选的参数为GL_COLOR_BUFFER_BIT,GL_DEPTH_BUFFER_BIT,GL_STENCIL_BUFFER_BIT分别标识颜色，深度，模板
		glfwSwapBuffers(mainWindow);//该函数会交换颜色缓冲（它是村塾GLFW窗口每一个像素颜色值的最大缓冲）
		glfwPollEvents();//检查有没有出发什么事件，比如键盘输入，鼠标移动等，更新窗口状态，并调用对应的回调函数
	}
	glfwTerminate();
	return 0;
}

void WindowSizeChangeCallback(GLFWwindow* window, int width, int height)
{
	glViewport(0, 0, width, height);
}
///--------------处理输入事件
void processInput(GLFWwindow* window)
{
	if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
	{
		glfwSetWindowShouldClose(window,true);
	}
}
void processVerticeslist() {
	unsigned int VBO;
	glGenBuffers(1,&VBO);//OpenGL允许绑定多个缓冲(类型不相同)
	glBindBuffer(GL_ARRAY_BUFFER,VBO);//把新建的缓冲绑定到GL_ARRAY_BUFFER上
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);//顶点数据复制到缓冲内存中
	//GL_STATIC_DRAW数据几乎不变 GL_DYNAMIC_DRAW数据会被改变很多 GL_STREAM_DRAW 数据每次绘制都会改变
}
//着色器设置一般流程
/*
unsigned int shaderName;//定义shader int
shaderName = glCreateShader(GL_VERTEX_SHADER);//创建指定shader类型
glShaderSource(shaderName,1,&shaderSource,NULL);//绑定shader和shaderSource
glCompileShader(shaderName);//编译shader
glGetShaderiv(shaderName,GL_COMPILE_STATUS,&success);//获取shader编译状态用于错误处理

linkeShader
*/
void processShader()
{
	int success;
	char infolog[512];
	//顶点着色器
	unsigned int vertexShader;
	vertexShader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
	glCompileShader(vertexShader);
	glGetShaderiv(vertexShader,GL_COMPILE_STATUS,&success);
	if (!success)
	{
		glGetShaderInfoLog(vertexShader,512,NULL,infolog);
		std::cout << "vertex shader error log:" << infolog << std::endl;
	}
	//片元着色器
	unsigned int fragmentShader;
	fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragmentShader,1,&fragmentShaderSource,NULL);
	glCompileShader(fragmentShader);
	glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
	if (!success)
	{
		glGetShaderInfoLog(fragmentShader, 512, NULL, infolog);
		std::cout << "vertex shader error log:" << infolog << std::endl;
	}
	//着色器程序
	unsigned int shaderProgram;
	shaderProgram = glCreateProgram();//创建shaderprogram
	glAttachShader(shaderProgram,vertexShader);//attach vertes shader
	glAttachShader(shaderProgram,fragmentShader);//attach fragment shader
	glLinkProgram(shaderProgram);//link shader
	glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
	if (!success)
	{
		glGetProgramInfoLog(shaderProgram,512,NULL,infolog);
	}
	glUseProgram(shaderProgram);
	glDeleteShader(vertexShader);
	glDeleteShader(fragmentShader);
}
```
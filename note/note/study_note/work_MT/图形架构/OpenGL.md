[最好的OpenGL教程之一\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1MJ411u7Bc/?spm_id_from=333.337.search-card.all.click&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)

[docs.gl](https://docs.gl/)

# P1：Why learning OpenGL
简单、跨平台，相比于其他 API：Vulkan、Direct3D.... 更适合新手，是它仅仅是一个由[Khronos组织](http://www.khronos.org/)制定并维护的规范(Specification)，可以访问 GPU 的图形 API，opengl 是由显卡制造产商实现的，在 GPU 驱动中


# P2：设置 OpenGL 和创建窗口

实现一个跨平台可以运行的窗口创建程序，`GLFW` 库可以帮助实现这一点

## GLFW 的安装
这里为了方便直接下载了 windows 的预编译版本
GLFW 官方网址： [An OpenGL library \| GLFW](https://www.glfw.org/)
右上角导航栏选择：Download，打开后 Windows 用户可选择 32-bit Windows binaries，linux 只能下载源码或者通过 `sudo apt install libglfw3-dev libglfw3` 安装 glfw
>选择 32-bit 在创建项目时，只能创建 32bit 的项目，至于选择 32bit的原因，作者没有明说，提到在 64bit 环境中也能运行 32bit 代码，猜测可能兼容性更好

打开后：找到 include 文件夹，和最新的 lib-vc 版本
![](http://cdn.ljc0606.cn/obsidian/202506252234202.png)

在项目目录下建立 Dependencies 文件夹，存放上述两个文件夹，删掉 lib 中的 `.dll` 和后缀 `dll` 的 lib 文件，这个用于 动态编译
配置：
```
项目属性 -> C/C++ -> 附加包含目录 -> 添加 $(SolutionDir)Dependencies\GLFW\include
项目属性 -> 链接器 -> 常规 -> 附加库目录 -> 添加: $(SolutionDir)Dependencies\GLFW\lib-vc2022
项目属性 -> 链接器 -> 输入 -> 附加依赖项 -> 添加: glfw3.lib
```

opengl 添加：
项目属性 -> 链接器 -> 输入 -> 附加依赖项 -> 添加: opengl32.lib
原视频中为了保持简洁，一个一个添加 lib，直接搜缺少的函数，然后在 MSDN 上能找到属于哪个lib
```
//直接默认lib
$(CoreLibraryDependencies);%(AdditionalDependencies);glfw3.lib;opengl32.lib
```

## 使用传统 opengl 绘制三角形

```
/* Render here */
glClear(GL_COLOR_BUFFER_BIT);

//Draw triangle
glBegin(GL_TRIANGLES);
glVertex2f(-0.5f, -0.5f);
glVertex2f(0.5f, -0.5f);
glVertex2f(0.0f, 0.5f);
glEnd();
```

# P3：使用现代 OpenGL
由于现代 OpenGL 是硬件驱动实现的非开源代码，需要用 GLEW 库来提供 OpenGL 的 api 接口，是 OpenGL 的扩展，其他库比如 GLUE 是一种 OpenGL 的特殊扩展。这里使用 GLEW。

## GLEW 配置
下载链接：[GLEW: The OpenGL Extension Wrangler Library](https://glew.sourceforge.net/)
在解压出来的文件夹，复制到 `Dependencies` 下面，重命名为 `GLEW`（方便索引），在文件夹中可以找到 `doc/`，有关于如何使用的 html，请阅读这份文档

```
项目属性 -> C/C++ -> 附加包含目录 -> 添加 $(SolutionDir)Dependencies\GLEW\include
项目属性 -> 链接器 -> 常规 -> 附加库目录 -> 添加: $(SolutionDir)Dependencies\GLEW\lib\Release\Win32;
项目属性 -> 链接器 -> 输入 -> 附加依赖项 -> 添加: glew32s.lib;
项目属性 -> C/C++ -> 预处理器 -> 预处理器定义 -> 添加：GLEW_STATIC
```

## Example

1. `#include <GL/glew.h>` 必须存在在任何 OpenGL 库的 include 之前
2. `glewinit()` 需要有 opengl 上下文环境

```
    /* Make the window's context current */
    glfwMakeContextCurrent(window);

	/* Initialize GLEW */
	if ( glewInit() != GLEW_OK)
    {
        std::cout << "Failed to initialize GLEW\n" << std::endl;
		return -1;
	}

    unsigned a;
    glGenBuffers(1, &a);

	std::cout << glGetString(GL_VERSION) << std::endl;   //print CPU opengl version
```

# P4：顶点缓冲区和绘制三角形
需要先进行一些设置：
1. 顶点缓冲区：一个在 GPU VRAM（Video RAM 上的 buffer），CPU 向 GPU 发出 Drawcall（绘制指令），然后使用着色器绘制
2. 着色器：是一段我们可以编写的，在显卡上运行的代码，告诉 GPU 应该如何绘制
图形管线将在后面的视频中提到

OpenGL 是一种状态机（state machine），在绘制之前需要先把数据传输到对应 buffer 作为状态，然后告诉 GPU 使用哪个 buffer 和哪个着色器进行绘制

## 使用现代 OpenGL 绘制三角形

```C++
    float position[6] = {
        -0.5f,-0.5f,
        0.5f, -0.5f,
        0.0f, 0.5f

    };

    unsigned int buffer;
    glGenBuffers(1, &buffer); //1 表示数量一个buffer buffer：返回buffer对应的id
    glBindBuffer(GL_ARRAY_BUFFER, buffer);
    glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), position, GL_STATIC_DRAW);   //define data type and uasge

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);

		glDrawArrays(GL_TRIANGLES, 0, 3); //draw triangles, 0 is the start index, 3 is the number of vertices
		.......
    }
```

OpenGL 是一种状态机，代码建议在上下文环境中，当建立 buffer 或者其他对象 object （顶点缓冲区、顶点数组、纹理、着色器等）时，会给出一个标识符 id 表示该对象，然后进行绑定或者选定对象时，使用这个 id

`glBufferData` 参数参考：
[glBufferData - OpenGL 3 - docs.gl](https://docs.gl/gl3/glBufferData)

# P5：顶点属性和内存布局

顶点表示图片上的一个点，包括位置、纹理坐标、颜色、法线等相关内容在内

当在 GPU 端使用时，需要告知 GPU 数据布局，这样 GPU 才能正确区分和解析数据

定义顶点数据属性和内存分布情况：
`glVertexAttribPointer` 比如有三个属性：坐标、纹理坐标和法线
index：属性的索引，例如：坐标在位置 1、纹理坐标在位置 2、法线在 3
size：每一个属性拥有多少元素，例如：2D 坐标为 2，3D 坐标为 3
type：数据类型：浮点等
normalized：归一化
stride： 每个顶点的字节数，例如：位置 12 个字节，纹理坐标 8 个字节，法线 12 个字节，一共 32 个字节
pointer：对于每个属性，相对于顶点开始位置的偏移量，例如：纹理坐标偏移量为 12

还需要使用 `glEnableVertexAttribArray` 启用
```c++
glEnableVertexAttribArray(0); 
//0 is the index of the vertex attribute, enable the vertex attribute array
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), (const void*)0); 
//0 is the index of the vertex attribute, 2 is the size of the attribute, GL_FLOAT is the data type, GL_FALSE means not normalized, 2 * sizeof(float) is the stride, (void*)0 is the offset

```

# P6：着色器原理
上面的代码已经可以实现一个三角形的绘制，虽然还没有实现着色器，但是当你没有着色器代码的时候，有些显卡会使用自带默认的着色器

**着色器**：是一个可以在 GPU 上运行的代码，更加复杂的场景需要对 GPU 进行编程
现阶段主要了解以下两种着色器：
Fragment shader 片段着色器，可能会执行很多次
Vertex shader 顶点着色器：目前会被执行三次(三个顶点)，告诉屏幕你希望在哪里绘制（where you want the position to be）

着色器的最终目的是决定像素颜色
简化 pipline 流程：Drawcall -> Vertex shader -> Fragment shader

Tips: 着色器也是以状态机的形式工作的，也需要进行启用

# P7：编写一个着色器
实现 vertex shader 和 fragment shader 使用 `createshader` 函数编译 shader 并且绑定到程序上，使用 `compileshader` 对着色器进行编译并且获得错误提示，使用 GLSL 语言描述着色器

两个编译 shader 函数
```c++
static unsigned int CompileShader(unsigned int type, const std::string& source)
{
	unsigned int id = glCreateShader(type); //create shader
	const char* src = source.c_str(); //convert string to c-string
	glShaderSource(id, 1, &src, nullptr); //set shader source
	glCompileShader(id); //compile shader

	//check for errors
	int result;
	glGetShaderiv(id, GL_COMPILE_STATUS, &result);
	if (result == GL_FALSE)
	{
        //get error message
		int length;
		glGetShaderiv(id, GL_INFO_LOG_LENGTH, &length);
		//char* message = new char[length];
		char* message = (char*)alloca(length * sizeof(char)); //use alloca to allocate memory on the stack
        glGetShaderInfoLog(id, length, &length, message);
		std::cout << "Failed to compile shader!" << (type == GL_VERTEX_SHADER ? "VERTEX" : "FRAGMENT") << "SHADER" << std::endl;
		std::cout << message << std::endl;
		delete[] message;
		glDeleteShader(id);
		return 0;
	}
	return id; //return shader id
}

static int CreateShader(const std::string& VertexShader, const std::string& FragmentShader)
{
	unsigned int program = glCreateProgram();
	unsigned int vs = CompileShader(GL_VERTEX_SHADER, VertexShader); //create vertex shader
	unsigned int fs = CompileShader(GL_FRAGMENT_SHADER, FragmentShader); //create fragment shader

    //link to one program
	glAttachShader(program, vs); //attach vertex shader
	glAttachShader(program, fs); //attach fragment shader
	glLinkProgram(program); //link program
	glValidateProgram(program); //validate program

	glDeleteShader(vs); //delete vertex shader
	glDeleteShader(fs); //delete fragment shader

	return program; //return program id
}
```

shader 程序描述，程序绑定和删除
```c++
//in main ：previous code
	glEnableVertexAttribArray(0); //0 is the index of the vertex attribute, enable the vertex attribute array
	glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), (const void*)0); //0 is the index of the vertex attribute, 2 is the size of the attribute, GL_FLOAT is the data type, GL_FALSE means not normalized, 2 * sizeof(float) is the stride, (void*)0 is the offset

//this code
	std::string vertexShader = R"(
		#version 330 core
		layout(location = 0) in vec4 position; //input vertex attribute
		void main()
		{
			gl_Position = position; //set the position of the vertex
		}
	)";

	std::string fragmentShader = R"(
		#version 330 core
		layout(location = 0) out vec4 color;  //output color of the fragment
		void main()
		{
			color = vec4(1.0, 0.0, 0.0, 1.0); //set the color of the fragment
		}
	)";

	/*
	#version 330 core  : using GLSL core means not using any deprecated function
	*/

	unsigned int shader = CreateShader(vertexShader, fragmentShader); //create shader program
	glUseProgram(shader); //use the shader program

	while......
	{
		........
	}

	glDeleteProgram(shader); //delete shader program
```

# P8：处理着色器

作者推荐：将顶点着色器和片段着色器写入一个文件中，这样更方便使用

## 将着色器使用文件形式导入
在 `ProjectDir` 下新建 `res/shader/Basic.shader`，将 shader 的内容导入
```GLSL
#shader vertex
#version 330 core

layout(location = 0) in vec4 position; //input vertex attribute

void main()
{
	gl_Position = position; //set the position of the vertex
}

#shader fragment
#version 330 core

layout(location = 0) out vec4 color;  //output color of the fragment

void main()
{
	color = vec4(1.0, 0.0, 0.0, 1.0); //set the color of the fragment
}

```

添加 `static ShaderProgramSource ParseShader(const std::string& filepath)` 函数
```c++
...
#include <fstream>
#include <string>
#include <sstream>

struct ShaderProgramSource
{ 
	std::string VertexSource;
	std::string FragmentSource;

};

static ShaderProgramSource ParseShader(const std::string& filepath)
{
	std::ifstream stream(filepath); //open file

	if (!stream.is_open()) {
		std::cerr << "ERROR: Could not open shader file at '"
			<< filepath << "'\n";
		// You could even `throw std::runtime_error(...)` here
	}

	std::string line;
	std::stringstream ss[2]; //create two stringstream for vertex and fragment shader

	enum class ShaderType
	{
		NONE = -1, VERTEX = 0, FRAGMENT = 1
	};
		
	ShaderType type = ShaderType::NONE; //initialize shader type to NONE

	while (getline(stream, line))
	{

		if (line.find("#shader") != std::string::npos)
		{
			if (line.find("vertex") != std::string::npos)
			{
				type = ShaderType::VERTEX; //set shader type to VERTEX
				std::cout << "Vertex shader find" << std::endl;
			}
			else if (line.find("fragment") != std::string::npos)
			{
				type = ShaderType::FRAGMENT; //set shader type to FRAGMENT
				std::cout << "Fragment shader find" << std::endl;
			}
		}
		else
		{
            if (type != ShaderType::NONE) // Check if the shader type is valid before appending
            {
               ss[(int)type] << line << '\n'; //append line to the corresponding stringstream
            }
		}
	}

	return { ss[0].str(), ss[1].str() }; //return the vertex and fragment shader source as a ShaderProgramSource struct
}

...

```

在 `main` 中输出
```C++
	ShaderProgramSource source = ParseShader("res/shader/Basic.shader"); //parse shader file
	std::cout << "VERTEX" << std::endl;
	std::cout << source.VertexSource << std::endl;
	std::cout << "FRAGMENT" << std::endl;
	std::cout << source.FragmentSource << std::endl;
```

检查路径：项目属性 -> 调试 -> 工作目录 -> $(ProjectDir)

# P9：索引缓冲区
 index buffer：reuse vertex
 
```c++
...
	float position[] = {
		-0.5f,-0.5f,
		0.5f, -0.5f,
		0.5f, 0.5f,

		0.5f, 0.5f,
		-0.5f,0.5f,
		-0.5f,-0.5f
	};
...
glBufferData(GL_ARRAY_BUFFER, 6 * 2 * sizeof(float), position, GL_STATIC_DRAW); 
...
glDrawArrays(GL_TRIANGLES, 0, 6); 
```

use index buffer to draw
```c++
float position[] = {
	-0.5f,-0.5f,  //0
	0.5f, -0.5f,  //1
	0.5f, 0.5f,   //2
	-0.5f,0.5f,   //3
};

unsigned int indices[] = {
	0, 1, 2, //first triangle
	2, 3, 0  //second triangle
};

unsigned int buffer;
glGenBuffers(1, &buffer); //1 表示数量一个buffer buffer：返回buffer对应的id
glBindBuffer(GL_ARRAY_BUFFER, buffer);
glBufferData(GL_ARRAY_BUFFER, 6 * 2 * sizeof(float), position, GL_STATIC_DRAW);   //define data type and uasge

glEnableVertexAttribArray(0); //0 is the index of the vertex attribute, enable the vertex attribute array
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), (const void*)0); //0 is the index of the vertex attribute, 2 is the size of the attribute, GL_FLOAT is the data type, GL_FALSE means not normalized, 2 * sizeof(float) is the stride, (void*)0 is the offset

unsigned int ibo; //index buffer object
glGenBuffers(1, &ibo); 
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ibo);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, 6 * sizeof(unsigned int), indices, GL_STATIC_DRAW);   //define data type and uasge


//...in while
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr); // draw rectangle with indices, nullptr means the offset is 0

```

>indices 只能使用 unsigned int 定义

# P10：错误处理
`glGetError` 会给出 OpenGL 出错的错误代码，但只会返回任意一个错误的代码，所以需要通过循环来不断打印所有错误信息
`glDebugMessageCallback` 更好，会给出错误信息而不是代码，并且给出建议，但是不是所有版本都兼容


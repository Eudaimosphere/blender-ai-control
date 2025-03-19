# Blender MCP服务器参考文档

## 简介

Blender MCP（Model Context Protocol）是一个允许AI模型（如Claude）直接与Blender进行交互和控制的系统。通过这种集成，AI可以执行3D建模、场景创建和操作等任务，实现无缝的人工智能辅助创作体验。

## 核心架构

该系统主要由两个组件构成：

1. **Blender插件（addon.py）**：在Blender内部创建一个套接字服务器，用于接收和执行命令
2. **MCP服务器（server.py）**：实现Model Context Protocol的Python服务器，连接到Blender插件

## 通信协议

系统通过TCP套接字使用基于JSON的简单协议进行通信：

- **命令**以包含`type`和可选`params`的JSON对象发送
- **响应**是包含`status`和`result`或`message`的JSON对象

## MCP命令类型

MCP服务器支持以下核心命令类型：

1. **execute_code**：在Blender的Python解释器中执行代码
   ```json
   {
     "type": "execute_code",
     "params": {
       "code": "import bpy; bpy.ops.mesh.primitive_cube_add()"
     }
   }
   ```

2. **get_scene_info**：获取当前场景的信息
   ```json
   {
     "type": "get_scene_info"
   }
   ```

3. **capture_viewport**：捕获当前视口的图像
   ```json
   {
     "type": "capture_viewport",
     "params": {
       "filepath": "/tmp/viewport.png"
     }
   }
   ```

4. **get_object_info**：获取特定对象的详细信息
   ```json
   {
     "type": "get_object_info",
     "params": {
       "object_name": "Cube"
     }
   }
   ```

## 操作流程

MCP控制Blender的典型流程如下：

1. **建立连接**：MCP服务器连接到Blender插件的套接字服务器
2. **初始化环境**：获取场景信息和当前状态
3. **执行操作**：发送一系列命令执行建模或动画操作
4. **获取反馈**：捕获视口图像和/或获取对象信息
5. **迭代优化**：基于反馈调整和优化模型

## 代码执行示例

以下是一些通过MCP服务器执行Blender Python API的示例：

### 创建基本几何体

```python
# 创建一个立方体
import bpy
bpy.ops.mesh.primitive_cube_add(size=2, location=(0, 0, 0))

# 创建一个球体
bpy.ops.mesh.primitive_uv_sphere_add(radius=1, location=(3, 0, 0))

# 创建一个圆柱体
bpy.ops.mesh.primitive_cylinder_add(radius=1, depth=2, location=(0, 3, 0))
```

### 材质操作

```python
# 为所选对象创建新材质
import bpy
obj = bpy.context.active_object
mat = bpy.data.materials.new(name="新材质")
mat.use_nodes = True
obj.data.materials.append(mat)

# 设置材质颜色
nodes = mat.node_tree.nodes
principled_bsdf = nodes.get("Principled BSDF")
if principled_bsdf:
    principled_bsdf.inputs["Base Color"].default_value = (1.0, 0.0, 0.0, 1.0)  # 红色
```

### 相机与渲染

```python
# 设置相机位置
import bpy
camera = bpy.data.objects["Camera"]
camera.location = (10, -10, 10)
camera.rotation_euler = (0.9, 0, 0.8)

# 设置渲染参数
bpy.context.scene.render.resolution_x = 1920
bpy.context.scene.render.resolution_y = 1080
bpy.context.scene.render.image_settings.file_format = 'PNG'
```

## 适配层实现

MCP服务器实现了一个适配层，使AI模型能更容易地使用高级命令：

### 命令翻译

适配层将AI的高级指令转换为精确的Blender Python代码：

- **创建对象**指令被翻译为对应的`bpy.ops.mesh.primitive_*_add()`调用
- **修改属性**指令转换为正确的对象属性访问路径
- **执行动画**指令被分解为关键帧插入系列

### 状态管理

适配层维护场景状态的内部表示，以提供更丰富的上下文：

- 跟踪创建的对象及其属性
- 维护操作历史以支持撤销和重做
- 提供上下文感知命令建议

## 技术实现细节

### 套接字通信

Blender插件使用Python的`socketserver`模块实现套接字服务器：

```python
import socketserver
import json

class BlenderRequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        data = self.request.recv(4096).decode('utf-8')
        command = json.loads(data)
        
        response = self.process_command(command)
        
        self.request.sendall(json.dumps(response).encode('utf-8'))
        
    def process_command(self, command):
        # 处理各种命令类型
        # ...
        return {"status": "success", "result": result}
```

### 代码执行安全

MCP服务器包含代码执行的安全机制：

- **沙箱执行**：在受限环境中执行代码
- **资源限制**：防止无限循环和过度资源消耗
- **访问控制**：限制对文件系统的访问

### 错误处理

系统实现了全面的错误捕获和报告机制：

```python
try:
    exec(code, globals_dict, locals_dict)
    response = {"status": "success", "result": result}
except Exception as e:
    response = {
        "status": "error", 
        "message": str(e),
        "type": type(e).__name__
    }
```

## 安装与配置

### 安装Blender插件

1. 下载Blender MCP插件文件`addon.py`
2. 在Blender中打开首选项 (Edit > Preferences > Add-ons)
3. 点击"Install..."并选择下载的插件文件
4. 启用插件

### 配置MCP服务器

1. 克隆GitHub仓库：
   ```
   git clone https://github.com/ahujasid/blender-mcp.git
   ```

2. 安装依赖：
   ```
   pip install -r requirements.txt
   ```

3. 配置连接设置：
   ```
   # config.py
   SERVER_HOST = "localhost"
   SERVER_PORT = 5000
   BLENDER_HOST = "localhost"
   BLENDER_PORT = 5001
   ```

4. 启动MCP服务器：
   ```
   python server.py
   ```

## 用户体验

使用MCP控制Blender提供了独特的体验优势：

### 自然语言交互

用户可以通过自然语言描述想要实现的3D模型或场景：

1. **描述性创作**：用户只需描述"创建一个红色的金属球体"，无需了解具体的Blender操作
2. **上下文理解**：系统理解上下文，如"将它移到左侧"会正确识别要操作的对象
3. **迭代改进**：用户可以通过简单的修改指令不断完善模型，如"让它更光滑"

### 交互式反馈循环

MCP系统创建了一个高效的人机协作循环：

1. **视觉确认**：执行操作后自动捕获视口图像，提供视觉反馈
2. **状态报告**：提供详细的操作结果和当前场景状态
3. **错误解释**：当操作失败时，系统提供易于理解的错误解释，而非技术错误堆栈

### 学习辅助

MCP系统也可作为Blender学习工具：

1. **代码生成**：将自然语言指令转换为实际的Python代码，帮助用户学习Blender API
2. **操作解释**：解释每个操作背后的原理和逻辑
3. **最佳实践**：推荐专业级的实现方式，而不仅仅是基础功能

## MCP的优势与局限性

### 优势

1. **降低技术门槛**：
   - 无需深入了解Blender的复杂界面和快捷键
   - 自然语言交互使3D创作更加直观
   - 减少学习曲线，加速入门过程

2. **提高创作效率**：
   - 快速实现复杂操作，减少重复性工作
   - 并行处理多个创作任务
   - 自动化常见工作流程

3. **创意增强**：
   - AI可以提供创意建议和替代方案
   - 探索用户可能不熟悉的技术和效果
   - 帮助突破创作瓶颈

### 局限性

1. **性能考量**：
   - 网络延迟可能影响实时交互体验
   - 大型场景的状态跟踪可能导致内存占用增加
   - 复杂操作序列需要优化以避免性能下降

2. **精确控制**：
   - 自然语言描述可能缺乏专业操作所需的精确度
   - 某些高度专业化的工作流程可能更适合直接手动操作
   - 可能需要混合使用MCP和直接界面交互

3. **安全性考虑**：
   - 执行外部代码存在潜在安全风险
   - 需要权限控制和资源限制
   - 对敏感项目可能需要额外的安全措施

## 使用案例

### 案例1：概念艺术家的快速原型

一位概念艺术家使用MCP系统快速将想法转化为3D模型：

```
用户: "我需要一个未来风格的飞行器，有流线型机身和两侧的弯曲翼"

MCP系统:
1. 创建流线型基础体（延长的椭球体）
2. 添加弯曲的翼部结构
3. 应用材质（金属质感，蓝色高光）
4. 添加发光引擎细节
5. 返回渲染视图供用户确认
```

艺术家只需简短的描述，就获得了3D模型的初步效果，然后可以通过更多的细节指示不断完善。

### 案例2：教育环境中的3D学习

一名Blender初学者在学习过程中使用MCP系统：

```
学生: "我想学习如何创建一个有纹理的木桌子"

MCP系统:
1. 创建桌面和桌腿几何体
2. 演示UV展开过程
3. 应用木纹材质
4. 提供每个步骤的Python代码解释
5. 展示替代方法和更高级的技术
```

学生不仅快速完成了模型创建，还学习了底层的技术原理，加深对Blender的理解。

### 案例3：视频制作团队的协作

视频制作团队使用MCP系统协调复杂场景的创建：

```
导演: "我们需要一个城市街道场景，有5栋不同风格的建筑，道路上有几辆车和行人"

MCP系统:
1. 创建基础城市网格
2. 生成多种风格的建筑物
3. 添加街道细节（道路、人行道、交通信号灯）
4. 放置车辆和行人
5. 设置环境光照和渲染参数
```

团队成员可以共同查看生成的场景，讨论修改点，并使用MCP系统迅速实现调整。

## 结论

Blender MCP服务器代表了3D创作工具与AI模型集成的强大潜力。通过建立一个标准化的协议和接口，它使AI模型能够理解、控制和操作Blender，从而为创作者提供前所未有的辅助体验。

MCP系统的核心价值在于：

1. **无缝集成**：将强大的AI能力直接融入现有的3D创作工作流中
2. **降低门槛**：使Blender的专业功能更容易被各级用户访问和利用
3. **效率提升**：通过自动化和智能辅助，大幅减少重复性工作
4. **创造力释放**：让创作者专注于创意构思，而非技术实现细节

随着AI技术的不断进步，MCP系统有望进一步发展，提供更智能、更直观的创作体验，最终可能重新定义3D设计和创作的工作方式。

未来发展方向包括：

- 更深入的上下文理解和预测能力
- 支持更复杂的多步骤创作流程
- 跨软件互操作性的扩展
- 协作功能的增强，支持多用户同时创作

Blender MCP服务器不仅是一个技术工具，更是创意与技术融合的桥梁，为创作者开启了一个充满可能性的新时代。 
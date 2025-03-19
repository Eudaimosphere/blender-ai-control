# Blender AI 控制研究

## 项目概述

这个研究项目探索了将自然语言处理与人工智能能力结合，用于控制Blender 3D软件的可能性。通过建立一个智能接口层，使艺术家和设计师能够使用自然语言描述来创建和操作3D模型，显著提升创作效率和降低使用门槛。

## 研究目标

1. 建立自然语言到Blender Python API的有效转换机制
2. 开发上下文感知的AI辅助系统，理解用户意图并执行相应操作
3. 创建交互式反馈循环，通过视觉反馈和迭代优化提升用户体验
4. 设计一个可扩展的架构，支持不断增加的功能和复杂度

## 项目结构

- `reference_python_blender.md` - Blender Python API的详细参考文档
- `reference_blender_mcp.md` - Blender MCP(Model Context Protocol)服务器实现参考

## 技术实现

本项目的核心是一个双层架构系统：

### 1. 语言理解层

- 使用大型语言模型(LLM)如Claude解析用户自然语言指令
- 将抽象意图转换为具体的操作步骤
- 维护对话上下文和操作历史

### 2. Blender执行层

- 基于Model Context Protocol的服务器连接
- Python代码生成和执行
- 状态管理和错误处理机制
- 视觉反馈捕获和处理

## 交互流程

1. **用户输入**：用户用自然语言描述想要创建或修改的3D内容
2. **意图解析**：AI分析输入并确定具体的Blender操作
3. **代码生成**：生成对应的Python代码
4. **执行操作**：在Blender中执行生成的代码
5. **状态捕获**：获取操作结果和当前场景状态
6. **视觉反馈**：提供渲染预览和操作确认
7. **迭代优化**：根据用户进一步的指示调整模型

## 示例指令转换

### 示例1: 创建一个红色的金属球体

**自然语言指令:**
```
创建一个红色的金属球体
```

**转换后的Python代码:**
```python
import bpy
bpy.ops.mesh.primitive_uv_sphere_add(radius=1, location=(0, 0, 0))
obj = bpy.context.active_object
mat = bpy.data.materials.new(name="红色金属")
mat.use_nodes = True
obj.data.materials.append(mat)
principled = mat.node_tree.nodes.get("Principled BSDF")
principled.inputs["Base Color"].default_value = (1.0, 0.1, 0.1, 1.0)
principled.inputs["Metallic"].default_value = 0.9
```

### 示例2: 移动和缩放对象

**自然语言指令:**
```
将它移到场景中心并放大两倍
```

**转换后的Python代码:**
```python
import bpy
obj = bpy.context.active_object
obj.location = (0, 0, 0)
obj.scale = (2, 2, 2)
```

## 未来发展方向

1. **多模态输入**：结合草图和语音输入增强指令精度
2. **协作创作**：支持多用户同时与AI系统交互创建内容
3. **学习与适应**：系统根据用户偏好和操作习惯优化交互体验
4. **插件生态系统**：为不同应用场景开发专用插件和工具集

## 研究状态

该项目目前处于研究阶段，主要关注基础概念验证和技术可行性分析。接下来的步骤包括：

1. 开发原型系统进行概念验证
2. 建立评估框架测量系统有效性
3. 进行用户研究收集实际使用反馈
4. 扩展支持更多Blender功能和操作类型

## 贡献指南

我们欢迎对此研究感兴趣的开发者、设计师和艺术家参与贡献。贡献可以包括：

- 改进自然语言理解模型
- 扩展Blender操作支持范围
- 开发更好的用户界面和交互模式
- 提供使用场景和测试案例

## 许可

该研究项目在MIT许可下发布。

## 联系方式

如果您对此研究有任何问题或建议，请联系项目维护者或提交Issue。 
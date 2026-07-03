---
name: chain-analysis
description: 分析代码调用链路，生成可导入 CoderTools IntelliJ 插件链路标注功能的 JSON 文件。当用户需要追踪代码执行路径、可视化调用链路、理解代码如何跨多个文件流转时使用。
license: MIT
metadata:
  author: zll
  version: "1.0"
---

分析代码调用链路，生成可导入 CoderTools 链路标注功能的 JSON 文件。

## 输出格式

生成名为 `chain-<链路名>.json` 的 JSON 文件，结构如下：

Windows 示例：
```json
{
  "chains": [
    {
      "name": "描述性链路名称",
      "color": "#3498DB",
      "steps": [
        {
          "filePath": "C:/Users/xxx/project/src/main/java/com/example/controller/UserController.java",
          "lineNumber": 25,
          "className": "UserController",
          "moduleName": "user-service",
          "remark": "接收HTTP请求"
        }
      ]
    }
  ]
}
```

macOS/Linux 示例：
```json
{
  "chains": [
    {
      "name": "描述性链路名称",
      "color": "#3498DB",
      "steps": [
        {
          "filePath": "/Users/xxx/project/src/main/java/com/example/controller/UserController.java",
          "lineNumber": 25,
          "className": "UserController",
          "moduleName": "user-service",
          "remark": "接收HTTP请求"
        }
      ]
    }
  ]
}
```

### 字段规则

| 字段 | 必填 | 说明 |
|------|------|------|
| `chains` | 是 | 链路对象数组 |
| `chains[].name` | 是 | 链路名称（如"用户登录流程"） |
| `chains[].color` | 否 | 十六进制颜色代码，缺省为 `#808080` |
| `chains[].steps` | 是 | 步骤对象数组（可以为空） |
| `steps[].filePath` | 是 | **项目绝对路径**，根据当前操作系统生成对应格式（Windows: `C:/Users/xxx/project/src/main/java/pkg/Foo.java`；macOS/Linux: `/Users/xxx/project/src/main/java/pkg/Foo.java`）。先读取项目根目录（`.git` 所在目录或 IDEA 项目根），拼接出完整路径 |
| `steps[].lineNumber` | 是 | 目标方法/类声明的行号（1-based） |
| `steps[].className` | 否 | 类名或函数名 |
| `steps[].moduleName` | 否 | 模块名或包名 |
| `steps[].remark` | 否 | 该步骤职责简述 |

**不要包含 `id` 字段** — 导入时会自动生成。

### 序列化规则

- 空值字段省略（`moduleName` 或 `remark` 为空时不输出该键）
- 使用 2-space JSON 缩进
- UTF-8 编码

## 分析策略

1. **识别入口点**：找到执行路径的起始点（Controller 处理方法、API 端点、main 方法、事件监听器等）

2. **追踪调用链路**：沿方法调用逐层追踪。对每个关键方法调用：
   - 记录**精确的文件绝对路径**（根据操作系统格式：Windows 用 `C:/...`，macOS/Linux 用 `/...`；路径分隔符统一使用 `/`）
   - 记录方法**声明位置**的精确行号（不是调用位置）
   - 用简短的 **remark** 描述该步骤的职责

3. **处理跨文件调用**：当调用跨入另一个文件时，记录目标文件和目标方法的声明行号。

4. **处理异步/事件驱动路径**：对 `@Async`、`@EventListener`、消息队列处理器等，作为单独步骤记录，在 remark 中注明异步性质。

5. **处理分支**：如果代码有条件分支（如成功路径 vs 错误路径），聚焦主路径/正常路径。如用户需要，可为替代路径创建单独链路。

6. **停止条件**：遇到以下情况停止追踪：
   - 调用到达框架/库边界（如 `JdbcTemplate.query()`、`RestTemplate.exchange()`）
   - 调用到达外部服务边界
   - 代码返回值且后续无有意义的处理

## 示例

给定以下 Spring MVC 代码：

```java
// UserController.java:25
@PostMapping("/login")
public Result login(@RequestBody LoginRequest req) {
    return userService.authenticate(req);
}

// UserService.java:42
public Result authenticate(LoginRequest req) {
    User user = userRepository.findByUsername(req.getUsername());
    if (user == null) throw new AuthException();
    // ...
}

// UserRepository.java:18
public User findByUsername(String username) {
    return jdbcTemplate.queryForObject(sql, mapper, username);
}
```

输出应为（Windows）：

```json
{
  "chains": [
    {
      "name": "用户登录流程",
      "color": "#E74C3C",
      "steps": [
        {
          "filePath": "C:/work/my-project/src/main/java/com/example/controller/UserController.java",
          "lineNumber": 25,
          "className": "UserController",
          "remark": "接收登录请求"
        },
        {
          "filePath": "C:/work/my-project/src/main/java/com/example/service/UserService.java",
          "lineNumber": 42,
          "className": "UserService",
          "remark": "验证用户凭证"
        },
        {
          "filePath": "C:/work/my-project/src/main/java/com/example/repository/UserRepository.java",
          "lineNumber": 18,
          "className": "UserRepository",
          "remark": "查询用户数据（到达数据层边界）"
        }
      ]
    }
  ]
}
```

输出应为（macOS/Linux）：

```json
{
  "chains": [
    {
      "name": "用户登录流程",
      "color": "#E74C3C",
      "steps": [
        {
          "filePath": "/Users/xxx/work/my-project/src/main/java/com/example/controller/UserController.java",
          "lineNumber": 25,
          "className": "UserController",
          "remark": "接收登录请求"
        },
        {
          "filePath": "/Users/xxx/work/my-project/src/main/java/com/example/service/UserService.java",
          "lineNumber": 42,
          "className": "UserService",
          "remark": "验证用户凭证"
        },
        {
          "filePath": "/Users/xxx/work/my-project/src/main/java/com/example/repository/UserRepository.java",
          "lineNumber": 18,
          "className": "UserRepository",
          "remark": "查询用户数据（到达数据层边界）"
        }
      ]
    }
  ]
}
```

## 导入到 CoderTools

生成 JSON 文件后，用户可通过以下步骤导入：

1. 在 IntelliJ IDEA 中下载并安装 **CoderTools** 插件（Plugins 市场搜索 "CoderTools"）
2. 在 IntelliJ IDEA 中打开 **Chains** 侧边栏
3. 点击 **导入** 按钮
4. 选择生成的 `.json` 文件
5. 链路将出现在侧边栏中，编辑器中会显示 gutter 标记

如果已存在同名链路，会自动添加后缀如 " (1)"。
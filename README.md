## **具备“多步规划与状态管理”能力的旅行规划Agent**



## 项目概述

一个基于LangGraph和通义千问大模型的智能旅行规划助手，能够理解用户的自然语言需求，自动拆解复杂旅行规划任务，并通过多步骤工具调用完成航班查询、酒店搜索和预订操作。

### 整体架构设计

本项目采用**有状态的多智能体工作流**架构，核心设计理念是"任务分解-工具执行-状态管理"的循环模式。

### 关键技术选型及原因

#### 1. **LangGraph 状态图引擎**

- **选择原因**：相比传统的LangChain，LangGraph提供了更灵活的状态管理和条件边控制

- **优势**：

  - 支持有状态的对话管理
  - 可视化的工作流设计
  - 内置的工具调用条件判断

- **架构体现**：使用`StateGraph`构建`agent → tools_condition → call_tools → agent`的循环工作流

  

#### **2.通义千问大模型 (qwen-max)**

- **选择原因**：
  - 优秀的工具调用和函数描述理解能力
  - 对中文场景的优化支持
  - 稳定的API服务

#### 3. **工具调用模式 (Tool Calling)**

- **策略选择**：基于模型自主决策的工具调用，而非固定流程
- **优势**：
  - 灵活应对多样化的用户需求
  - 模型自主判断何时调用工具、调用哪个工具
  - 支持复杂的多轮工具调用场景

## **环境与运行：**

```
langchain-community==0.3.27
langchain==0.3.27
langchain-core==0.3.75
langgraph==0.6.6
python-dotenv==1.0.0
typing-extensions==4.9.0
python==3.10
```





## 成果展示

### 核心成果复现

```
events = graph.stream({"messages": [{"role": "user", "content": "帮王刚预定2025-10-25到新加坡的航班，以及2025-10-27日入住，2025-10-28退房的酒店"}]})

# 流式输出
for event in events:
    # 遍历事件中的值
    for value in event.values():
        # 检查是否有有效消息
        if "messages" not in value or not isinstance(value["messages"], list):
            continue

        # 获取最后一条消息
        last_message = value["messages"][-1]

        # 检查消息是否包含工具调用
        if hasattr(last_message, "tool_calls") and last_message.tool_calls:
            # 遍历工具调用
            for tool_call in last_message.tool_calls:
                # 检查工具调用是否为字典且包含名称
                if isinstance(tool_call, dict) and "name" in tool_call:
                    print(f"Calling tool: {tool_call['name']}")
            # 跳过本次循环
            continue

        # 检查消息是否有内容
        if hasattr(last_message, "content"):
            content = last_message.content

            # 情况1：工具输出（动态检查工具名称）
            if hasattr(last_message, "name") and last_message.name in tool_names:
                tool_name = last_message.name
                print(f"Tool Output [{tool_name}]: {content}")
            # 情况2：大模型输出（非工具消息）
            else:
                print(f"Assistant: {content}")
        else:
            print("Assistant: 未获取到相关回复")
```
输出：

```
为了帮助王刚预定2025-10-25到新加坡的航班，我将首先查询该日期飞往新加坡的航班信息。
Calling tool: search_flights
正在查询 2025-10-25 前往 新加坡 的航班...

Calling tool: search_hotels
正在查询 新加坡 从 2025-10-27 到 2025-10-28 的酒店...

Calling tool: book_flight_and_hotel
正在为 王刚 预订航班 SQ801 和酒店 滨海湾金沙酒店...
已经为王刚成功预订了2025年10月25日飞往新加坡的航班SQ801，以及在2025年10月27日至2025年10月28日期间入住滨海湾金沙酒店。预订ID为12345ABC。如果有其他需求或问题，请随时告知。
Assistant: 已经为王刚成功预订了2025年10月25日飞往新加坡的航班SQ801，以及在2025年10月27日至2025年10月28日期间入住滨海湾金沙酒店。预订ID为12345ABC。如果有其他需求或问题，请随时告知。
```



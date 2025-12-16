# AI Agent 概述

## 什么是 AI Agent

**AI Agent（智能体/智能代理）** 是一种能够自主感知环境、做出决策并执行任务的人工智能系统。与传统的问答式 AI 不同，Agent 具备自主性、目标导向性和持续学习能力。

```
┌─────────────────────────────────────────────────────────────┐
│                        AI Agent                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  感知   │→│  规划   │→│  决策   │→│  执行   │        │
│  │Perceive │  │  Plan   │  │ Decide  │  │ Execute │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
│       ↑                                       │              │
│       └───────────── 反馈循环 ─────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### Agent vs 传统 AI 对比

| 特性 | 传统 AI（ChatGPT） | AI Agent |
|------|-------------------|----------|
| 交互方式 | 一问一答 | 自主执行多步骤任务 |
| 任务复杂度 | 单轮对话 | 复杂、多步骤任务 |
| 工具使用 | 有限或无 | 可调用多种外部工具 |
| 记忆能力 | 上下文窗口限制 | 长期记忆 + 短期记忆 |
| 自主性 | 被动响应 | 主动规划和执行 |
| 环境感知 | 仅文本输入 | 多模态感知 |

---

## AI Agent 核心架构

一个完整的 AI Agent 系统通常包含以下核心组件：

```
Agent = LLM（大脑） + Memory（记忆） + Tools（工具） + Planning（规划）
```

### 1. LLM - 大脑

LLM 是 Agent 的核心推理引擎，负责理解任务、做出决策。

```javascript
/**
 * Agent 的 LLM 核心
 */
class AgentBrain {
  constructor(model = 'gpt-4') {
    this.model = model;
  }

  /**
   * 理解用户意图
   */
  async understand(input) {
    const response = await this.callLLM({
      messages: [
        {
          role: 'system',
          content: `你是一个任务分析专家。分析用户的请求，提取：
            1. 用户的核心目标
            2. 需要完成的子任务
            3. 可能需要的工具
            4. 潜在的挑战`
        },
        { role: 'user', content: input }
      ]
    });

    return JSON.parse(response);
  }

  /**
   * 决策下一步行动
   */
  async decide(context) {
    const { goal, currentState, availableTools, history } = context;

    const response = await this.callLLM({
      messages: [
        {
          role: 'system',
          content: `你是一个决策专家。根据当前状态决定下一步行动。
            可用工具：${availableTools.map(t => t.name).join(', ')}

            输出格式：
            {
              "action": "工具名称或 'respond'",
              "params": { 工具参数 },
              "reasoning": "选择该行动的原因"
            }`
        },
        { role: 'user', content: JSON.stringify({ goal, currentState, history }) }
      ]
    });

    return JSON.parse(response);
  }

  async callLLM(options) {
    // 调用 LLM API
  }
}
```

### 2. Memory - 记忆系统

Agent 需要记忆系统来存储和检索信息，分为短期记忆和长期记忆。

```javascript
/**
 * Agent 记忆系统
 */
class AgentMemory {
  constructor() {
    // 短期记忆：当前对话上下文
    this.shortTerm = [];

    // 长期记忆：向量数据库存储
    this.longTerm = new VectorStore();

    // 工作记忆：当前任务状态
    this.working = new Map();
  }

  /**
   * 添加到短期记忆
   */
  addToShortTerm(message) {
    this.shortTerm.push({
      ...message,
      timestamp: Date.now()
    });

    // 维护窗口大小
    if (this.shortTerm.length > 20) {
      // 将旧记忆转移到长期记忆
      const old = this.shortTerm.shift();
      this.addToLongTerm(old);
    }
  }

  /**
   * 添加到长期记忆
   */
  async addToLongTerm(content) {
    const embedding = await this.getEmbedding(content);
    await this.longTerm.add({
      content,
      embedding,
      metadata: {
        timestamp: Date.now(),
        type: content.type || 'general'
      }
    });
  }

  /**
   * 检索相关记忆
   */
  async recall(query, topK = 5) {
    // 从短期记忆获取最近的上下文
    const recent = this.shortTerm.slice(-5);

    // 从长期记忆检索相关内容
    const relevant = await this.longTerm.search(query, topK);

    return {
      recent,
      relevant: relevant.map(r => r.content)
    };
  }

  /**
   * 更新工作记忆
   */
  updateWorking(key, value) {
    this.working.set(key, value);
  }

  /**
   * 获取工作记忆
   */
  getWorking(key) {
    return this.working.get(key);
  }

  async getEmbedding(text) {
    // 调用 Embedding API
  }
}
```

### 3. Tools - 工具系统

Tools 扩展了 Agent 的能力边界，使其能够与外部世界交互。

```javascript
/**
 * 工具基类
 */
class Tool {
  constructor(name, description, parameters) {
    this.name = name;
    this.description = description;
    this.parameters = parameters;
  }

  /**
   * 执行工具
   */
  async execute(params) {
    throw new Error('Must implement execute method');
  }

  /**
   * 获取工具 Schema（用于 LLM Function Calling）
   */
  getSchema() {
    return {
      name: this.name,
      description: this.description,
      parameters: {
        type: 'object',
        properties: this.parameters,
        required: Object.keys(this.parameters)
      }
    };
  }
}

/**
 * 网页搜索工具
 */
class WebSearchTool extends Tool {
  constructor() {
    super(
      'web_search',
      '搜索互联网获取最新信息',
      {
        query: { type: 'string', description: '搜索关键词' },
        num_results: { type: 'number', description: '返回结果数量' }
      }
    );
  }

  async execute({ query, num_results = 5 }) {
    // 调用搜索 API（如 SerpAPI、Bing API）
    const response = await fetch(`https://api.search.com/search?q=${encodeURIComponent(query)}&n=${num_results}`);
    const results = await response.json();

    return results.map(r => ({
      title: r.title,
      snippet: r.snippet,
      url: r.url
    }));
  }
}

/**
 * 代码执行工具
 */
class CodeExecutionTool extends Tool {
  constructor() {
    super(
      'execute_code',
      '执行 JavaScript 代码并返回结果',
      {
        code: { type: 'string', description: '要执行的 JavaScript 代码' }
      }
    );
  }

  async execute({ code }) {
    try {
      // 在沙箱环境中执行代码
      const sandbox = new Sandbox();
      const result = await sandbox.run(code);
      return { success: true, result };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }
}

/**
 * 文件操作工具
 */
class FileOperationTool extends Tool {
  constructor() {
    super(
      'file_operation',
      '读取或写入文件',
      {
        operation: { type: 'string', enum: ['read', 'write', 'list'] },
        path: { type: 'string', description: '文件路径' },
        content: { type: 'string', description: '写入内容（仅 write 操作）' }
      }
    );
  }

  async execute({ operation, path, content }) {
    switch (operation) {
      case 'read':
        return await fs.readFile(path, 'utf-8');
      case 'write':
        await fs.writeFile(path, content);
        return { success: true };
      case 'list':
        return await fs.readdir(path);
      default:
        throw new Error(`Unknown operation: ${operation}`);
    }
  }
}

/**
 * 工具注册中心
 */
class ToolRegistry {
  constructor() {
    this.tools = new Map();
  }

  register(tool) {
    this.tools.set(tool.name, tool);
  }

  get(name) {
    return this.tools.get(name);
  }

  getAll() {
    return Array.from(this.tools.values());
  }

  getSchemas() {
    return this.getAll().map(tool => tool.getSchema());
  }
}
```

### 4. Planning - 规划系统

Planning 负责将复杂任务分解为可执行的子任务，并制定执行策略。

```javascript
/**
 * 任务规划器
 */
class TaskPlanner {
  constructor(llm) {
    this.llm = llm;
  }

  /**
   * 分解任务为子任务
   */
  async decompose(goal) {
    const response = await this.llm.callLLM({
      messages: [
        {
          role: 'system',
          content: `你是一个任务规划专家。将复杂任务分解为可执行的子任务。

            输出格式：
            {
              "goal": "总目标",
              "tasks": [
                {
                  "id": 1,
                  "name": "任务名称",
                  "description": "任务描述",
                  "dependencies": [], // 依赖的任务 ID
                  "tools": [] // 可能需要的工具
                }
              ],
              "execution_order": [1, 2, 3] // 执行顺序
            }`
        },
        { role: 'user', content: goal }
      ]
    });

    return JSON.parse(response);
  }

  /**
   * 生成执行计划
   */
  async createPlan(goal, context) {
    const { availableTools, constraints, preferences } = context;

    // 1. 分解任务
    const decomposition = await this.decompose(goal);

    // 2. 验证工具可用性
    const validatedTasks = decomposition.tasks.map(task => ({
      ...task,
      tools: task.tools.filter(t => availableTools.includes(t))
    }));

    // 3. 优化执行顺序
    const optimizedOrder = this.optimizeOrder(validatedTasks, constraints);

    return {
      goal: decomposition.goal,
      tasks: validatedTasks,
      executionOrder: optimizedOrder,
      estimatedSteps: validatedTasks.length
    };
  }

  /**
   * 优化执行顺序（拓扑排序）
   */
  optimizeOrder(tasks, constraints) {
    const graph = new Map();
    const inDegree = new Map();

    // 构建依赖图
    tasks.forEach(task => {
      graph.set(task.id, []);
      inDegree.set(task.id, task.dependencies.length);
    });

    tasks.forEach(task => {
      task.dependencies.forEach(dep => {
        graph.get(dep).push(task.id);
      });
    });

    // 拓扑排序
    const queue = [];
    const result = [];

    inDegree.forEach((degree, id) => {
      if (degree === 0) queue.push(id);
    });

    while (queue.length > 0) {
      const current = queue.shift();
      result.push(current);

      graph.get(current).forEach(next => {
        inDegree.set(next, inDegree.get(next) - 1);
        if (inDegree.get(next) === 0) {
          queue.push(next);
        }
      });
    }

    return result;
  }
}
```

---

## Agent 工作流程

### ReAct 模式

ReAct（Reasoning + Acting）是最常用的 Agent 执行模式，交替进行推理和行动。

```
Thought → Action → Observation → Thought → Action → Observation → ... → Final Answer
```

```javascript
/**
 * ReAct Agent 实现
 */
class ReActAgent {
  constructor(config) {
    this.llm = new AgentBrain(config.model);
    this.memory = new AgentMemory();
    this.tools = new ToolRegistry();
    this.maxIterations = config.maxIterations || 10;

    // 注册工具
    config.tools.forEach(tool => this.tools.register(tool));
  }

  /**
   * 执行任务
   */
  async run(task) {
    const history = [];
    let iteration = 0;

    while (iteration < this.maxIterations) {
      iteration++;

      // 1. Thought - 推理当前状态
      const thought = await this.think(task, history);
      history.push({ type: 'thought', content: thought });

      // 检查是否完成
      if (thought.isComplete) {
        return {
          success: true,
          answer: thought.finalAnswer,
          history
        };
      }

      // 2. Action - 决定并执行行动
      const action = await this.act(thought);
      history.push({ type: 'action', content: action });

      // 3. Observation - 观察执行结果
      const observation = await this.observe(action);
      history.push({ type: 'observation', content: observation });

      // 更新记忆
      this.memory.addToShortTerm({
        thought,
        action,
        observation
      });
    }

    return {
      success: false,
      error: 'Max iterations reached',
      history
    };
  }

  /**
   * 思考阶段
   */
  async think(task, history) {
    const prompt = `
任务：${task}

历史记录：
${history.map(h => `${h.type}: ${JSON.stringify(h.content)}`).join('\n')}

可用工具：
${this.tools.getSchemas().map(s => `- ${s.name}: ${s.description}`).join('\n')}

基于以上信息，分析当前状态并决定下一步：
1. 当前进度如何？
2. 还需要做什么？
3. 是否已经完成任务？

输出格式：
{
  "analysis": "当前状态分析",
  "nextStep": "下一步计划",
  "isComplete": false,
  "finalAnswer": null
}

如果任务已完成，设置 isComplete 为 true，并提供 finalAnswer。
`;

    const response = await this.llm.callLLM({
      messages: [{ role: 'user', content: prompt }]
    });

    return JSON.parse(response);
  }

  /**
   * 行动阶段
   */
  async act(thought) {
    const prompt = `
基于以下分析，选择要执行的工具：

分析：${thought.analysis}
计划：${thought.nextStep}

可用工具：
${JSON.stringify(this.tools.getSchemas(), null, 2)}

输出格式：
{
  "tool": "工具名称",
  "params": { 工具参数 },
  "reason": "选择原因"
}
`;

    const response = await this.llm.callLLM({
      messages: [{ role: 'user', content: prompt }]
    });

    const action = JSON.parse(response);

    // 执行工具
    const tool = this.tools.get(action.tool);
    if (tool) {
      action.result = await tool.execute(action.params);
    } else {
      action.result = { error: `Tool ${action.tool} not found` };
    }

    return action;
  }

  /**
   * 观察阶段
   */
  async observe(action) {
    return {
      tool: action.tool,
      input: action.params,
      output: action.result,
      timestamp: Date.now()
    };
  }
}

// 使用示例
const agent = new ReActAgent({
  model: 'gpt-4',
  maxIterations: 10,
  tools: [
    new WebSearchTool(),
    new CodeExecutionTool(),
    new FileOperationTool()
  ]
});

const result = await agent.run('帮我查找最新的 React 19 新特性，并写一个总结');
console.log(result);
```

### Plan-and-Execute 模式

先制定完整计划，再按计划执行。

```javascript
/**
 * Plan-and-Execute Agent
 */
class PlanExecuteAgent {
  constructor(config) {
    this.llm = new AgentBrain(config.model);
    this.planner = new TaskPlanner(this.llm);
    this.tools = new ToolRegistry();

    config.tools.forEach(tool => this.tools.register(tool));
  }

  async run(goal) {
    // 1. 规划阶段
    console.log('📋 Creating plan...');
    const plan = await this.planner.createPlan(goal, {
      availableTools: this.tools.getAll().map(t => t.name),
      constraints: {},
      preferences: {}
    });

    console.log('Plan:', plan);

    // 2. 执行阶段
    const results = [];

    for (const taskId of plan.executionOrder) {
      const task = plan.tasks.find(t => t.id === taskId);
      console.log(`🔄 Executing: ${task.name}`);

      const result = await this.executeTask(task, results);
      results.push({ taskId, task, result });

      console.log(`✅ Completed: ${task.name}`);
    }

    // 3. 汇总结果
    const summary = await this.summarize(goal, results);

    return {
      goal,
      plan,
      results,
      summary
    };
  }

  async executeTask(task, previousResults) {
    // 根据任务选择合适的工具执行
    const tool = this.tools.get(task.tools[0]);

    if (!tool) {
      // 无工具可用，使用 LLM 直接处理
      return await this.llm.callLLM({
        messages: [
          { role: 'user', content: `执行任务：${task.description}\n上下文：${JSON.stringify(previousResults)}` }
        ]
      });
    }

    // 让 LLM 生成工具参数
    const params = await this.generateToolParams(task, previousResults);
    return await tool.execute(params);
  }

  async generateToolParams(task, context) {
    const response = await this.llm.callLLM({
      messages: [
        {
          role: 'system',
          content: `根据任务生成工具参数。任务：${task.description}\n上下文：${JSON.stringify(context)}`
        }
      ]
    });

    return JSON.parse(response);
  }

  async summarize(goal, results) {
    const response = await this.llm.callLLM({
      messages: [
        {
          role: 'user',
          content: `
目标：${goal}

执行结果：
${results.map(r => `- ${r.task.name}: ${JSON.stringify(r.result)}`).join('\n')}

请总结任务完成情况，并给出最终答案。
`
        }
      ]
    });

    return response;
  }
}
```

---

## Agent 类型分类

### 按自主程度分类

| 类型 | 自主程度 | 特点 | 应用场景 |
|------|---------|------|----------|
| 简单 Agent | 低 | 执行预定义的简单任务 | 客服问答、简单查询 |
| 工具型 Agent | 中 | 根据需要调用工具 | 数据分析、代码生成 |
| 自主 Agent | 高 | 自主规划和执行复杂任务 | 项目管理、研究分析 |
| 协作 Agent | 高 | 多 Agent 协作完成任务 | 复杂项目、模拟仿真 |

### 按应用领域分类

```javascript
// 1. 对话型 Agent
class ConversationalAgent {
  // 专注于自然对话，理解用户意图
}

// 2. 任务执行型 Agent
class TaskExecutionAgent {
  // 专注于完成特定任务
}

// 3. 数据分析型 Agent
class DataAnalysisAgent {
  // 专注于数据处理和分析
}

// 4. 代码生成型 Agent
class CodeGenerationAgent {
  // 专注于代码生成和优化
}

// 5. 研究型 Agent
class ResearchAgent {
  // 专注于信息收集和研究
}
```

---

## Multi-Agent 系统

多个 Agent 协作完成复杂任务。

```javascript
/**
 * Multi-Agent 协调器
 */
class MultiAgentOrchestrator {
  constructor() {
    this.agents = new Map();
    this.messageQueue = [];
  }

  /**
   * 注册 Agent
   */
  registerAgent(name, agent, capabilities) {
    this.agents.set(name, {
      agent,
      capabilities,
      status: 'idle'
    });
  }

  /**
   * 分配任务给合适的 Agent
   */
  async assignTask(task) {
    // 分析任务需要的能力
    const requiredCapabilities = await this.analyzeTask(task);

    // 找到合适的 Agent
    const suitableAgents = this.findSuitableAgents(requiredCapabilities);

    if (suitableAgents.length === 0) {
      throw new Error('No suitable agent found for this task');
    }

    // 分配给最合适的 Agent
    const selectedAgent = suitableAgents[0];
    selectedAgent.status = 'busy';

    const result = await selectedAgent.agent.run(task);
    selectedAgent.status = 'idle';

    return result;
  }

  /**
   * 协作执行复杂任务
   */
  async collaborativeExecution(goal) {
    // 1. 任务分解
    const subtasks = await this.decomposeGoal(goal);

    // 2. 分配子任务
    const assignments = subtasks.map(task => ({
      task,
      agent: this.findBestAgent(task)
    }));

    // 3. 并行执行独立任务
    const independentTasks = assignments.filter(a => !a.task.dependencies.length);
    const dependentTasks = assignments.filter(a => a.task.dependencies.length > 0);

    // 执行独立任务
    const independentResults = await Promise.all(
      independentTasks.map(a => a.agent.agent.run(a.task))
    );

    // 按依赖顺序执行依赖任务
    const allResults = [...independentResults];
    for (const assignment of dependentTasks) {
      const result = await assignment.agent.agent.run(assignment.task);
      allResults.push(result);
    }

    // 4. 汇总结果
    return this.aggregateResults(goal, allResults);
  }

  findSuitableAgents(capabilities) {
    return Array.from(this.agents.values())
      .filter(a =>
        capabilities.every(cap => a.capabilities.includes(cap)) &&
        a.status === 'idle'
      );
  }

  findBestAgent(task) {
    // 简单策略：找第一个能处理的 Agent
    return this.findSuitableAgents(task.requiredCapabilities)[0];
  }

  async analyzeTask(task) {
    // 使用 LLM 分析任务需要的能力
    return ['search', 'analyze'];
  }

  async decomposeGoal(goal) {
    // 任务分解逻辑
    return [];
  }

  async aggregateResults(goal, results) {
    // 结果汇总逻辑
    return { goal, results };
  }
}

// 使用示例
const orchestrator = new MultiAgentOrchestrator();

// 注册专门的 Agent
orchestrator.registerAgent('researcher', new ResearchAgent(), ['search', 'summarize']);
orchestrator.registerAgent('coder', new CodeAgent(), ['code', 'debug']);
orchestrator.registerAgent('writer', new WriterAgent(), ['write', 'edit']);

// 协作完成复杂任务
const result = await orchestrator.collaborativeExecution(
  '研究 React 19 新特性，并写一篇技术博客，包含代码示例'
);
```

---

## 常见面试题

### 概念理解题

**1. 什么是 AI Agent？它与 ChatGPT 有什么区别？**

**答案要点：**
- AI Agent 是能自主感知、决策、执行的 AI 系统
- ChatGPT 是问答式 AI，一问一答
- Agent 可以使用工具、有记忆、能规划
- Agent 能完成多步骤复杂任务

**2. 解释 Agent 的核心组件：LLM、Memory、Tools、Planning**

**答案要点：**
- LLM：推理引擎，负责理解和决策
- Memory：记忆系统，存储上下文和历史
- Tools：工具集，扩展能力边界
- Planning：规划器，分解和调度任务

**3. 什么是 ReAct 模式？它是如何工作的？**

**答案要点：**
- ReAct = Reasoning + Acting
- 交替进行推理和行动
- 流程：Thought → Action → Observation → ...
- 每次行动后观察结果，再进行下一轮推理

### 设计题

**4. 设计一个能够自动处理客户退款请求的 Agent**

```javascript
// 答案示例
class RefundAgent {
  constructor() {
    this.tools = [
      new OrderQueryTool(),      // 查询订单
      new RefundPolicyTool(),    // 查询退款政策
      new RefundProcessTool(),   // 处理退款
      new NotificationTool()     // 发送通知
    ];
  }

  async processRefund(request) {
    // 1. 理解请求
    const intent = await this.understand(request);

    // 2. 查询订单信息
    const order = await this.tools[0].execute({ orderId: intent.orderId });

    // 3. 检查退款政策
    const eligible = await this.tools[1].execute({
      orderDate: order.date,
      reason: intent.reason
    });

    if (!eligible) {
      return { success: false, reason: '不符合退款政策' };
    }

    // 4. 处理退款
    const refund = await this.tools[2].execute({
      orderId: order.id,
      amount: order.amount
    });

    // 5. 通知用户
    await this.tools[3].execute({
      userId: order.userId,
      message: `退款 ${refund.amount} 元已处理`
    });

    return { success: true, refund };
  }
}
```

**5. 如何实现 Agent 的记忆系统？**

**答案要点：**
- 短期记忆：对话上下文，数组/队列存储
- 长期记忆：向量数据库存储，语义检索
- 工作记忆：当前任务状态，Map 存储
- 记忆管理：定期清理、重要性排序

---

## 总结

### AI Agent 核心要点

1. **架构组成**：LLM + Memory + Tools + Planning
2. **执行模式**：ReAct（推理-行动）、Plan-and-Execute（规划-执行）
3. **工具系统**：扩展 Agent 能力的关键
4. **记忆系统**：短期记忆 + 长期记忆 + 工作记忆
5. **Multi-Agent**：多 Agent 协作完成复杂任务

### 关键技术

- Function Calling / Tool Use
- 向量数据库与语义检索
- 任务规划与分解
- 错误处理与重试机制
- 安全性与权限控制

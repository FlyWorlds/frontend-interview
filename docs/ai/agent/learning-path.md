# AI Agent 入门与学习路径

## 概述

本文为前端开发者提供一条系统的 AI Agent 学习路径，从基础知识到实战应用，帮助你快速入门 AI Agent 开发。

---

## 前端转 AI Agent 的优势

作为前端开发者，你已经具备了很多转型 AI Agent 开发的优势：

| 前端技能 | AI Agent 应用 |
|---------|--------------|
| API 调用经验 | LLM API 集成、流式响应处理 |
| 状态管理 | Agent 状态流转、记忆管理 |
| 异步编程 | 并发任务处理、工具调用 |
| UI/UX 设计 | Agent 交互界面、对话 UI |
| 工程化能力 | Agent 应用部署、监控 |
| Node.js 经验 | 后端服务开发、工具实现 |

---

## 学习路径总览

```
┌──────────────────────────────────────────────────────────────────┐
│                      AI Agent 学习路径                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  第一阶段：基础知识（1-2 周）                                       │
│  ├─ LLM 基础概念                                                  │
│  ├─ API 调用与流式响应                                            │
│  └─ Prompt Engineering                                           │
│                                                                   │
│  第二阶段：核心技术（2-3 周）                                       │
│  ├─ Agent 核心概念                                                │
│  ├─ 工具调用（Function Calling）                                  │
│  ├─ 向量数据库与 RAG                                              │
│  └─ 记忆系统设计                                                  │
│                                                                   │
│  第三阶段：框架实践（2-3 周）                                       │
│  ├─ LangChain.js                                                 │
│  ├─ Vercel AI SDK                                                │
│  └─ 其他 Agent 框架                                               │
│                                                                   │
│  第四阶段：项目实战（持续）                                         │
│  ├─ 聊天机器人                                                    │
│  ├─ 文档问答系统                                                  │
│  ├─ 代码助手                                                      │
│  └─ 自动化工作流                                                  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 第一阶段：基础知识

### 1.1 LLM 基础概念

#### 什么是 LLM

大语言模型（Large Language Model）是基于 Transformer 架构的深度学习模型，通过训练海量文本数据来理解和生成人类语言。

**核心概念：**

```javascript
// 1. Token - LLM 的基本处理单位
const text = "Hello, World!";
// 英文：约 1 token ≈ 4 个字符
// 中文：约 1 token ≈ 1-2 个汉字

// 2. Context Window - 上下文窗口
const models = {
  'gpt-3.5-turbo': 16384,    // 16K tokens
  'gpt-4': 128000,            // 128K tokens
  'claude-3': 200000,         // 200K tokens
};

// 3. Temperature - 控制输出随机性
const settings = {
  temperature: 0,    // 确定性输出，适合代码生成
  temperature: 0.7,  // 平衡创意和一致性
  temperature: 1.5,  // 高创意，可能不连贯
};
```

#### 主流 LLM 对比

| 模型 | 提供商 | 特点 | 适用场景 |
|------|--------|------|----------|
| GPT-4 | OpenAI | 综合能力强 | 通用任务 |
| GPT-4o | OpenAI | 多模态、速度快 | 实时应用 |
| Claude 3 | Anthropic | 长上下文、安全 | 文档分析 |
| Gemini | Google | 多模态原生 | 图文处理 |
| 文心一言 | 百度 | 中文优化 | 中文应用 |
| 通义千问 | 阿里 | 开源可部署 | 私有化部署 |
| Llama 3 | Meta | 开源免费 | 本地部署 |

### 1.2 API 调用与流式响应

#### 基础 API 调用

```javascript
/**
 * OpenAI Chat Completions API
 */
async function chat(messages) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
    },
    body: JSON.stringify({
      model: 'gpt-4',
      messages: messages,
      temperature: 0.7,
      max_tokens: 2000
    })
  });

  const data = await response.json();
  return data.choices[0].message.content;
}

// 使用示例
const response = await chat([
  { role: 'system', content: '你是一个有帮助的助手' },
  { role: 'user', content: '什么是 AI Agent？' }
]);
```

#### 流式响应处理

```javascript
/**
 * 流式响应 - 逐字输出效果
 */
async function streamChat(messages, onChunk) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
    },
    body: JSON.stringify({
      model: 'gpt-4',
      messages: messages,
      stream: true  // 开启流式
    })
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = '';

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split('\n');
    buffer = lines.pop() || '';

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') continue;

        try {
          const parsed = JSON.parse(data);
          const content = parsed.choices[0]?.delta?.content;
          if (content) {
            onChunk(content);
          }
        } catch (e) {
          // 忽略解析错误
        }
      }
    }
  }
}

// 使用示例
let fullResponse = '';
await streamChat(
  [{ role: 'user', content: '讲个故事' }],
  (chunk) => {
    fullResponse += chunk;
    process.stdout.write(chunk); // 实时输出
  }
);
```

#### 前端流式渲染

```vue
<template>
  <div class="chat">
    <div class="messages">
      <div v-for="msg in messages" :key="msg.id" :class="msg.role">
        <div v-html="renderMarkdown(msg.content)"></div>
        <span v-if="msg.isStreaming" class="cursor">|</span>
      </div>
    </div>
    <input v-model="input" @keyup.enter="send" :disabled="isLoading" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { marked } from 'marked';

const messages = ref([]);
const input = ref('');
const isLoading = ref(false);

function renderMarkdown(content) {
  return marked(content);
}

async function send() {
  if (!input.value.trim() || isLoading.value) return;

  const userMessage = {
    id: Date.now(),
    role: 'user',
    content: input.value
  };

  messages.value.push(userMessage);

  const assistantMessage = {
    id: Date.now() + 1,
    role: 'assistant',
    content: '',
    isStreaming: true
  };

  messages.value.push(assistantMessage);
  input.value = '';
  isLoading.value = true;

  try {
    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        messages: messages.value.slice(0, -1).map(m => ({
          role: m.role,
          content: m.content
        }))
      })
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value);
      // 解析 SSE 数据
      const lines = chunk.split('\n');
      for (const line of lines) {
        if (line.startsWith('data: ') && line !== 'data: [DONE]') {
          try {
            const data = JSON.parse(line.slice(6));
            const content = data.choices[0]?.delta?.content;
            if (content) {
              assistantMessage.content += content;
            }
          } catch (e) {}
        }
      }
    }
  } finally {
    assistantMessage.isStreaming = false;
    isLoading.value = false;
  }
}
</script>
```

### 1.3 Prompt Engineering

#### 基础技巧

```javascript
/**
 * Prompt Engineering 最佳实践
 */

// 1. 明确角色
const systemPrompt = `
你是一位资深的前端架构师，专精于：
- React/Vue 框架
- 性能优化
- 工程化实践
- 代码审查

请用专业但易懂的方式回答问题。
`;

// 2. 提供上下文
const contextPrompt = `
【项目背景】
- 技术栈：React 18 + TypeScript + Vite
- 问题：首页加载时间 5.2s
- 目标：优化到 2s 以内

【当前代码】
\`\`\`tsx
${code}
\`\`\`

请分析性能瓶颈并给出优化方案。
`;

// 3. 结构化输出
const structuredPrompt = `
分析用户反馈，按以下 JSON 格式输出：

{
  "sentiment": "positive|negative|neutral",
  "category": "bug|feature|question",
  "priority": "high|medium|low",
  "summary": "一句话总结",
  "suggestedAction": "建议的处理方式"
}

用户反馈：${feedback}
`;

// 4. Few-Shot Learning
const fewShotPrompt = `
将自然语言转换为 SQL 查询：

示例 1:
输入: "查询所有年龄大于 30 的用户"
输出: SELECT * FROM users WHERE age > 30;

示例 2:
输入: "统计每个部门的员工数量"
输出: SELECT department, COUNT(*) FROM employees GROUP BY department;

现在转换:
输入: "${userInput}"
输出:
`;

// 5. Chain of Thought
const cotPrompt = `
问题：如何设计一个高可用的前端错误监控系统？

请按以下步骤思考：
1. 需要捕获哪些类型的错误？
2. 如何收集错误信息？
3. 如何上报和存储？
4. 如何分析和告警？
5. 有哪些现有方案可以参考？

一步步分析并给出设计方案。
`;
```

#### 高级技巧

```javascript
/**
 * 高级 Prompt 技巧
 */

// 1. Self-Consistency（自我一致性）
async function selfConsistency(question, numSamples = 3) {
  const answers = await Promise.all(
    Array(numSamples).fill(null).map(() =>
      callLLM({
        messages: [{ role: 'user', content: question }],
        temperature: 0.7  // 允许一定随机性
      })
    )
  );

  // 找出最一致的答案
  return findMostConsistent(answers);
}

// 2. ReAct Prompting
const reactPrompt = `
使用以下格式回答问题：

Thought: 分析问题，思考下一步
Action: 选择工具 [search, calculate, lookup]
Action Input: 工具输入
Observation: 工具返回结果
... (重复上述步骤)
Thought: 得出最终答案
Final Answer: 最终答案

问题：${question}
`;

// 3. Tree of Thoughts
async function treeOfThoughts(problem) {
  // 生成多个思考路径
  const paths = await generateThoughtPaths(problem, 3);

  // 评估每个路径
  const evaluatedPaths = await Promise.all(
    paths.map(async (path) => ({
      path,
      score: await evaluatePath(path)
    }))
  );

  // 选择最优路径继续
  const bestPath = evaluatedPaths.sort((a, b) => b.score - a.score)[0];

  // 沿最优路径深入
  return await expandPath(bestPath.path);
}
```

---

## 第二阶段：核心技术

### 2.1 Agent 核心概念

请参考 [AI Agent 概述](/ai/agent/) 了解 Agent 的完整架构。

### 2.2 Function Calling（工具调用）

Function Calling 是让 LLM 调用外部工具的关键技术。

```javascript
/**
 * OpenAI Function Calling 示例
 */
const tools = [
  {
    type: 'function',
    function: {
      name: 'get_weather',
      description: '获取指定城市的天气信息',
      parameters: {
        type: 'object',
        properties: {
          city: {
            type: 'string',
            description: '城市名称，如：北京、上海'
          },
          unit: {
            type: 'string',
            enum: ['celsius', 'fahrenheit'],
            description: '温度单位'
          }
        },
        required: ['city']
      }
    }
  },
  {
    type: 'function',
    function: {
      name: 'search_web',
      description: '搜索互联网获取信息',
      parameters: {
        type: 'object',
        properties: {
          query: {
            type: 'string',
            description: '搜索关键词'
          },
          num_results: {
            type: 'number',
            description: '返回结果数量'
          }
        },
        required: ['query']
      }
    }
  }
];

// 工具实现
const toolImplementations = {
  get_weather: async ({ city, unit = 'celsius' }) => {
    // 调用天气 API
    const response = await fetch(
      `https://api.weather.com/v1/weather?city=${encodeURIComponent(city)}&unit=${unit}`
    );
    return await response.json();
  },

  search_web: async ({ query, num_results = 5 }) => {
    // 调用搜索 API
    const response = await fetch(
      `https://api.search.com/search?q=${encodeURIComponent(query)}&n=${num_results}`
    );
    return await response.json();
  }
};

/**
 * 带工具调用的对话
 */
async function chatWithTools(messages) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
    },
    body: JSON.stringify({
      model: 'gpt-4',
      messages,
      tools,
      tool_choice: 'auto'  // 让模型决定是否调用工具
    })
  });

  const data = await response.json();
  const message = data.choices[0].message;

  // 检查是否需要调用工具
  if (message.tool_calls) {
    const toolResults = await Promise.all(
      message.tool_calls.map(async (toolCall) => {
        const functionName = toolCall.function.name;
        const args = JSON.parse(toolCall.function.arguments);

        // 执行工具
        const result = await toolImplementations[functionName](args);

        return {
          tool_call_id: toolCall.id,
          role: 'tool',
          content: JSON.stringify(result)
        };
      })
    );

    // 将工具结果添加到消息中，继续对话
    return chatWithTools([
      ...messages,
      message,
      ...toolResults
    ]);
  }

  return message.content;
}

// 使用示例
const response = await chatWithTools([
  { role: 'user', content: '北京今天天气怎么样？' }
]);
// LLM 会自动调用 get_weather 工具获取天气信息
```

### 2.3 向量数据库与 RAG

#### Embedding 基础

```javascript
/**
 * 文本向量化
 */
async function getEmbedding(text) {
  const response = await fetch('https://api.openai.com/v1/embeddings', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
    },
    body: JSON.stringify({
      model: 'text-embedding-3-small',
      input: text
    })
  });

  const data = await response.json();
  return data.data[0].embedding;  // 返回 1536 维向量
}

/**
 * 计算余弦相似度
 */
function cosineSimilarity(a, b) {
  const dotProduct = a.reduce((sum, val, i) => sum + val * b[i], 0);
  const magnitudeA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
  const magnitudeB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
  return dotProduct / (magnitudeA * magnitudeB);
}
```

#### 简单向量存储

```javascript
/**
 * 简单的内存向量存储
 */
class SimpleVectorStore {
  constructor() {
    this.documents = [];
  }

  async add(text, metadata = {}) {
    const embedding = await getEmbedding(text);
    this.documents.push({
      id: Date.now().toString(),
      text,
      embedding,
      metadata
    });
  }

  async search(query, topK = 5) {
    const queryEmbedding = await getEmbedding(query);

    const results = this.documents
      .map(doc => ({
        ...doc,
        similarity: cosineSimilarity(queryEmbedding, doc.embedding)
      }))
      .sort((a, b) => b.similarity - a.similarity)
      .slice(0, topK);

    return results;
  }
}
```

#### RAG 实现

```javascript
/**
 * RAG（检索增强生成）实现
 */
class RAGSystem {
  constructor() {
    this.vectorStore = new SimpleVectorStore();
  }

  /**
   * 添加文档到知识库
   */
  async addDocuments(documents) {
    for (const doc of documents) {
      // 文档分块
      const chunks = this.chunkDocument(doc.content);

      for (const chunk of chunks) {
        await this.vectorStore.add(chunk, {
          title: doc.title,
          source: doc.source
        });
      }
    }
  }

  /**
   * 文档分块
   */
  chunkDocument(text, chunkSize = 500, overlap = 50) {
    const chunks = [];
    for (let i = 0; i < text.length; i += chunkSize - overlap) {
      chunks.push(text.slice(i, i + chunkSize));
    }
    return chunks;
  }

  /**
   * 问答
   */
  async query(question) {
    // 1. 检索相关文档
    const relevantDocs = await this.vectorStore.search(question, 3);

    // 2. 构建增强 prompt
    const context = relevantDocs
      .map(doc => `[来源: ${doc.metadata.title}]\n${doc.text}`)
      .join('\n\n---\n\n');

    const prompt = `
基于以下参考资料回答问题。如果资料中没有相关信息，请明确说明。

参考资料：
${context}

问题：${question}

要求：
1. 只基于提供的资料回答
2. 引用信息来源
3. 如果不确定，说明不确定性
`;

    // 3. 调用 LLM 生成答案
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
      },
      body: JSON.stringify({
        model: 'gpt-4',
        messages: [
          { role: 'system', content: '你是一个知识助手，只基于提供的资料回答问题。' },
          { role: 'user', content: prompt }
        ]
      })
    });

    const data = await response.json();
    return {
      answer: data.choices[0].message.content,
      sources: relevantDocs.map(d => d.metadata)
    };
  }
}

// 使用示例
const rag = new RAGSystem();

// 添加知识库
await rag.addDocuments([
  {
    title: 'React 文档',
    content: 'React 是一个用于构建用户界面的 JavaScript 库...',
    source: 'react.dev'
  },
  {
    title: 'Vue 文档',
    content: 'Vue 是一个渐进式 JavaScript 框架...',
    source: 'vuejs.org'
  }
]);

// 查询
const result = await rag.query('React 和 Vue 有什么区别？');
console.log(result.answer);
```

### 2.4 记忆系统设计

```javascript
/**
 * Agent 记忆系统
 */
class MemorySystem {
  constructor() {
    // 短期记忆：对话历史
    this.shortTermMemory = [];

    // 长期记忆：向量存储
    this.longTermMemory = new SimpleVectorStore();

    // 摘要记忆：压缩的历史
    this.summaryMemory = [];

    // 配置
    this.config = {
      shortTermLimit: 10,    // 短期记忆消息数限制
      summaryThreshold: 20,  // 触发摘要的消息数
    };
  }

  /**
   * 添加消息到记忆
   */
  async addMessage(message) {
    this.shortTermMemory.push({
      ...message,
      timestamp: Date.now()
    });

    // 检查是否需要压缩记忆
    if (this.shortTermMemory.length > this.config.summaryThreshold) {
      await this.compressMemory();
    }
  }

  /**
   * 压缩记忆（生成摘要）
   */
  async compressMemory() {
    // 取出旧消息
    const oldMessages = this.shortTermMemory.splice(
      0,
      this.shortTermMemory.length - this.config.shortTermLimit
    );

    // 生成摘要
    const summary = await this.generateSummary(oldMessages);

    // 存入摘要记忆
    this.summaryMemory.push({
      summary,
      timestamp: Date.now(),
      messageCount: oldMessages.length
    });

    // 存入长期记忆
    await this.longTermMemory.add(summary, {
      type: 'conversation_summary',
      timestamp: Date.now()
    });
  }

  /**
   * 生成对话摘要
   */
  async generateSummary(messages) {
    const conversationText = messages
      .map(m => `${m.role}: ${m.content}`)
      .join('\n');

    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
      },
      body: JSON.stringify({
        model: 'gpt-3.5-turbo',
        messages: [
          {
            role: 'system',
            content: '简洁地总结以下对话的要点，保留关键信息。'
          },
          {
            role: 'user',
            content: conversationText
          }
        ],
        max_tokens: 500
      })
    });

    const data = await response.json();
    return data.choices[0].message.content;
  }

  /**
   * 获取相关记忆
   */
  async getRelevantMemory(query) {
    // 获取短期记忆
    const recentMessages = this.shortTermMemory.slice(-5);

    // 搜索长期记忆
    const relevantLongTerm = await this.longTermMemory.search(query, 3);

    // 获取相关摘要
    const relevantSummaries = this.summaryMemory.slice(-2);

    return {
      recent: recentMessages,
      longTerm: relevantLongTerm.map(r => r.text),
      summaries: relevantSummaries.map(s => s.summary)
    };
  }

  /**
   * 构建带记忆的 prompt
   */
  async buildPromptWithMemory(userMessage) {
    const memory = await this.getRelevantMemory(userMessage);

    let contextParts = [];

    // 添加摘要
    if (memory.summaries.length > 0) {
      contextParts.push(
        '【历史摘要】\n' + memory.summaries.join('\n')
      );
    }

    // 添加相关长期记忆
    if (memory.longTerm.length > 0) {
      contextParts.push(
        '【相关记忆】\n' + memory.longTerm.join('\n')
      );
    }

    // 构建消息
    const messages = [
      {
        role: 'system',
        content: `你是一个有记忆的助手。以下是相关的历史信息：\n\n${contextParts.join('\n\n')}`
      },
      ...memory.recent,
      { role: 'user', content: userMessage }
    ];

    return messages;
  }
}
```

---

## 第三阶段：框架实践

详见 [技术栈与框架](/ai/agent/tech-stack) 文档。

---

## 第四阶段：项目实战

详见 [实战案例](/ai/agent/practice) 文档。

---

## 需要学习的知识和技能

### 编程语言

| 语言 | 重要程度 | 用途 |
|------|---------|------|
| JavaScript/TypeScript | 必须 | 前端开发、Node.js 后端 |
| Python | 推荐 | AI/ML 生态、数据处理 |
| SQL | 推荐 | 数据查询、向量数据库 |

### 核心技术栈

```
前端基础
├── React / Vue
├── TypeScript
├── 状态管理
└── API 调用

AI/LLM
├── Prompt Engineering
├── Function Calling
├── Embedding
└── RAG

后端服务
├── Node.js / Next.js
├── API 设计
├── 数据库
└── 部署运维

Agent 专项
├── LangChain.js
├── Vercel AI SDK
├── 向量数据库
└── Agent 框架
```

### 学习资源

**官方文档（必读）**
- [OpenAI API 文档](https://platform.openai.com/docs)
- [Anthropic Claude 文档](https://docs.anthropic.com)
- [LangChain.js 文档](https://js.langchain.com)
- [Vercel AI SDK 文档](https://sdk.vercel.ai)

**课程推荐**
- 吴恩达《ChatGPT Prompt Engineering》
- 吴恩达《Building Systems with ChatGPT》
- 吴恩达《LangChain for LLM Application Development》

**实践平台**
- [Coze](https://www.coze.com) - 低代码 Agent 平台
- [Dify](https://dify.ai) - 开源 LLM 应用开发平台
- [FlowiseAI](https://flowiseai.com) - 可视化 LLM 编排

**社区资源**
- GitHub 热门 Agent 项目
- Hugging Face 模型和数据集
- AI 相关 Discord 社区

---

## 学习建议

### 1. 循序渐进

```
Week 1-2: 基础 API 调用，熟悉 Prompt Engineering
Week 3-4: 学习 Function Calling，实现简单工具
Week 5-6: 学习 RAG，实现文档问答
Week 7-8: 学习 Agent 框架，实现完整 Agent
Week 9+:  项目实战，持续学习
```

### 2. 边学边做

每学一个概念，立即动手实践：
- API 调用 → 实现一个聊天界面
- Function Calling → 添加天气查询功能
- RAG → 实现文档问答
- Agent → 实现自动化任务

### 3. 关注前沿

- 订阅 AI 相关 Newsletter
- 关注主要 AI 公司的博客
- 参与开源社区讨论
- 尝试新发布的模型和工具

### 4. 建立作品集

- GitHub 开源项目
- 技术博客文章
- Demo 演示项目
- 参与开源贡献

---

## 常见问题

**Q: 前端转 AI Agent 难度大吗？**

A: 前端开发者有很好的基础。主要需要学习 LLM API 使用、Prompt Engineering 和一些 AI 特有概念（如 Embedding、RAG）。技术栈大部分可以复用。

**Q: 需要学习 Python 吗？**

A: 建议学习。虽然 JavaScript/TypeScript 可以完成大部分工作，但 Python 在 AI/ML 领域生态更丰富，很多教程和示例都是 Python 的。

**Q: 本地部署 LLM 需要什么配置？**

A: 取决于模型大小。小模型（7B 参数）需要 8GB+ 显存，大模型需要更高配置。可以先使用 API，等有需求再考虑本地部署。

**Q: 如何选择 LLM 提供商？**

A: 根据需求选择：
- 通用任务：GPT-4 或 Claude
- 中文优先：文心一言、通义千问
- 成本敏感：GPT-3.5、开源模型
- 私有部署：Llama、通义千问

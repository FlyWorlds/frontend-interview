# AI Agent 技术栈与框架

## 概述

本文详细介绍 AI Agent 开发中常用的技术栈、框架和工具，帮助你选择适合的技术方案。

---

## 技术栈全景图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AI Agent 技术栈                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                        应用层                                  │   │
│  │  聊天机器人 │ 文档问答 │ 代码助手 │ 自动化工作流 │ 数据分析      │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              ↑                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                      Agent 框架层                              │   │
│  │  LangChain │ Vercel AI SDK │ AutoGPT │ CrewAI │ LlamaIndex    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              ↑                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                        能力层                                  │   │
│  │  Function Calling │ RAG │ Memory │ Planning │ Multi-Agent     │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              ↑                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                       基础设施层                               │   │
│  │  LLM API │ 向量数据库 │ 工具 API │ 存储 │ 计算资源              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## LLM API 服务

### 主流 LLM API 对比

| 服务商 | 模型 | 特点 | 价格（输入/输出 per 1K tokens） |
|--------|------|------|-------------------------------|
| OpenAI | GPT-4o | 多模态、速度快 | $5 / $15 |
| OpenAI | GPT-4 Turbo | 128K 上下文 | $10 / $30 |
| OpenAI | GPT-3.5 Turbo | 性价比高 | $0.5 / $1.5 |
| Anthropic | Claude 3 Opus | 最强推理 | $15 / $75 |
| Anthropic | Claude 3 Sonnet | 均衡性能 | $3 / $15 |
| Anthropic | Claude 3 Haiku | 速度最快 | $0.25 / $1.25 |
| Google | Gemini Pro | 多模态 | $0.5 / $1.5 |
| 百度 | 文心一言 | 中文优化 | 按量计费 |
| 阿里 | 通义千问 | 开源可部署 | 按量计费 |

### OpenAI SDK

```javascript
/**
 * OpenAI SDK 使用示例
 */
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// 基础对话
async function chat(messages) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages,
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}

// 流式对话
async function* streamChat(messages) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4',
    messages,
    stream: true,
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) {
      yield content;
    }
  }
}

// Function Calling
async function chatWithFunctions(messages, functions) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages,
    tools: functions.map(f => ({
      type: 'function',
      function: f
    })),
    tool_choice: 'auto',
  });

  return completion.choices[0].message;
}

// Embedding
async function getEmbedding(text) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: text,
  });

  return response.data[0].embedding;
}

// 图像生成
async function generateImage(prompt) {
  const response = await openai.images.generate({
    model: 'dall-e-3',
    prompt,
    n: 1,
    size: '1024x1024',
  });

  return response.data[0].url;
}

// 图像理解
async function analyzeImage(imageUrl, question) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4-vision-preview',
    messages: [
      {
        role: 'user',
        content: [
          { type: 'text', text: question },
          { type: 'image_url', image_url: { url: imageUrl } },
        ],
      },
    ],
  });

  return response.choices[0].message.content;
}
```

### Anthropic SDK (Claude)

```javascript
/**
 * Anthropic Claude SDK 使用示例
 */
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// 基础对话
async function chat(messages) {
  const response = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages,
  });

  return response.content[0].text;
}

// 流式对话
async function* streamChat(messages) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages,
  });

  for await (const event of stream) {
    if (event.type === 'content_block_delta' &&
        event.delta.type === 'text_delta') {
      yield event.delta.text;
    }
  }
}

// 工具使用
async function chatWithTools(messages, tools) {
  const response = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    tools,
    messages,
  });

  return response;
}

// 示例工具定义
const tools = [
  {
    name: 'get_weather',
    description: '获取指定城市的天气信息',
    input_schema: {
      type: 'object',
      properties: {
        city: {
          type: 'string',
          description: '城市名称'
        }
      },
      required: ['city']
    }
  }
];
```

---

## Agent 开发框架

### LangChain.js

LangChain 是最流行的 LLM 应用开发框架，提供完整的 Agent 开发工具链。

```bash
npm install langchain @langchain/openai @langchain/community
```

#### 基础使用

```javascript
/**
 * LangChain.js 基础示例
 */
import { ChatOpenAI } from '@langchain/openai';
import { HumanMessage, SystemMessage } from '@langchain/core/messages';
import { StringOutputParser } from '@langchain/core/output_parsers';
import { ChatPromptTemplate } from '@langchain/core/prompts';

// 创建模型
const model = new ChatOpenAI({
  modelName: 'gpt-4',
  temperature: 0.7,
});

// 基础对话
async function chat(userMessage) {
  const messages = [
    new SystemMessage('你是一个有帮助的助手'),
    new HumanMessage(userMessage),
  ];

  const response = await model.invoke(messages);
  return response.content;
}

// 使用 Prompt Template
const promptTemplate = ChatPromptTemplate.fromMessages([
  ['system', '你是一个 {language} 专家'],
  ['human', '{question}'],
]);

async function askExpert(language, question) {
  const chain = promptTemplate.pipe(model).pipe(new StringOutputParser());

  return await chain.invoke({
    language,
    question,
  });
}
```

#### Chain 组合

```javascript
/**
 * LangChain Chain 组合
 */
import { RunnableSequence, RunnablePassthrough } from '@langchain/core/runnables';
import { StructuredOutputParser } from 'langchain/output_parsers';
import { z } from 'zod';

// 结构化输出
const outputSchema = z.object({
  summary: z.string().describe('内容摘要'),
  keywords: z.array(z.string()).describe('关键词列表'),
  sentiment: z.enum(['positive', 'negative', 'neutral']).describe('情感倾向'),
});

const parser = StructuredOutputParser.fromZodSchema(outputSchema);

const analysisChain = RunnableSequence.from([
  {
    text: new RunnablePassthrough(),
    format_instructions: () => parser.getFormatInstructions(),
  },
  ChatPromptTemplate.fromTemplate(`
    分析以下文本：

    {text}

    {format_instructions}
  `),
  model,
  parser,
]);

async function analyzeText(text) {
  return await analysisChain.invoke(text);
}

// 多步骤 Chain
const multiStepChain = RunnableSequence.from([
  // 步骤 1：提取关键信息
  ChatPromptTemplate.fromTemplate('从以下文本提取关键信息：\n{input}'),
  model,
  new StringOutputParser(),

  // 步骤 2：生成摘要
  (keyInfo) => ({ keyInfo }),
  ChatPromptTemplate.fromTemplate('基于关键信息生成摘要：\n{keyInfo}'),
  model,
  new StringOutputParser(),
]);
```

#### Agent 实现

```javascript
/**
 * LangChain Agent 实现
 */
import { AgentExecutor, createOpenAIFunctionsAgent } from 'langchain/agents';
import { DynamicTool, DynamicStructuredTool } from '@langchain/core/tools';
import { pull } from 'langchain/hub';

// 定义工具
const searchTool = new DynamicTool({
  name: 'web_search',
  description: '搜索互联网获取最新信息',
  func: async (query) => {
    // 实现搜索逻辑
    const results = await searchWeb(query);
    return JSON.stringify(results);
  },
});

const calculatorTool = new DynamicStructuredTool({
  name: 'calculator',
  description: '进行数学计算',
  schema: z.object({
    expression: z.string().describe('数学表达式'),
  }),
  func: async ({ expression }) => {
    try {
      const result = eval(expression);
      return `计算结果: ${result}`;
    } catch (e) {
      return `计算错误: ${e.message}`;
    }
  },
});

// 创建 Agent
async function createAgent() {
  const tools = [searchTool, calculatorTool];

  // 从 LangChain Hub 获取 prompt
  const prompt = await pull('hwchase17/openai-functions-agent');

  const agent = await createOpenAIFunctionsAgent({
    llm: model,
    tools,
    prompt,
  });

  const agentExecutor = new AgentExecutor({
    agent,
    tools,
    verbose: true,  // 打印执行过程
  });

  return agentExecutor;
}

// 使用 Agent
async function runAgent(input) {
  const agent = await createAgent();
  const result = await agent.invoke({
    input,
  });

  return result.output;
}
```

#### RAG 实现

```javascript
/**
 * LangChain RAG 实现
 */
import { OpenAIEmbeddings } from '@langchain/openai';
import { MemoryVectorStore } from 'langchain/vectorstores/memory';
import { RecursiveCharacterTextSplitter } from 'langchain/text_splitter';
import { createRetrievalChain } from 'langchain/chains/retrieval';
import { createStuffDocumentsChain } from 'langchain/chains/combine_documents';

// 创建 RAG 系统
async function createRAGSystem(documents) {
  // 1. 文档分块
  const textSplitter = new RecursiveCharacterTextSplitter({
    chunkSize: 1000,
    chunkOverlap: 200,
  });

  const splits = await textSplitter.splitDocuments(documents);

  // 2. 创建向量存储
  const embeddings = new OpenAIEmbeddings();
  const vectorStore = await MemoryVectorStore.fromDocuments(
    splits,
    embeddings
  );

  // 3. 创建检索器
  const retriever = vectorStore.asRetriever({
    k: 4,  // 返回 4 个最相关的文档
  });

  // 4. 创建问答 Chain
  const prompt = ChatPromptTemplate.fromTemplate(`
    基于以下上下文回答问题。如果上下文中没有相关信息，请说明你不知道。

    上下文：
    {context}

    问题：{input}

    回答：
  `);

  const documentChain = await createStuffDocumentsChain({
    llm: model,
    prompt,
  });

  const retrievalChain = await createRetrievalChain({
    combineDocsChain: documentChain,
    retriever,
  });

  return retrievalChain;
}

// 使用 RAG
async function queryRAG(ragChain, question) {
  const result = await ragChain.invoke({
    input: question,
  });

  return {
    answer: result.answer,
    sources: result.context.map(doc => doc.metadata),
  };
}
```

#### 记忆系统

```javascript
/**
 * LangChain 记忆系统
 */
import { BufferMemory, ConversationSummaryMemory } from 'langchain/memory';
import { ConversationChain } from 'langchain/chains';

// 缓冲记忆（保存完整对话）
const bufferMemory = new BufferMemory({
  returnMessages: true,
  memoryKey: 'chat_history',
});

// 摘要记忆（自动生成对话摘要）
const summaryMemory = new ConversationSummaryMemory({
  llm: model,
  returnMessages: true,
});

// 创建带记忆的对话 Chain
const conversationChain = new ConversationChain({
  llm: model,
  memory: bufferMemory,
  verbose: true,
});

// 对话
async function chatWithMemory(input) {
  const response = await conversationChain.call({ input });
  return response.response;
}

// 向量记忆（语义检索历史）
import { VectorStoreRetrieverMemory } from 'langchain/memory';

const vectorMemory = new VectorStoreRetrieverMemory({
  vectorStoreRetriever: vectorStore.asRetriever(1),
  memoryKey: 'relevant_history',
});
```

### Vercel AI SDK

Vercel AI SDK 是专为前端设计的 AI 开发工具包，特别适合 Next.js 项目。

```bash
npm install ai @ai-sdk/openai @ai-sdk/anthropic
```

#### 基础使用

```javascript
/**
 * Vercel AI SDK 基础示例
 */
import { generateText, streamText, generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';
import { z } from 'zod';

// 生成文本
async function chat(prompt) {
  const result = await generateText({
    model: openai('gpt-4'),
    prompt,
  });

  return result.text;
}

// 流式生成
async function* streamChat(prompt) {
  const result = await streamText({
    model: openai('gpt-4'),
    prompt,
  });

  for await (const chunk of result.textStream) {
    yield chunk;
  }
}

// 结构化输出
async function generateStructured(prompt) {
  const result = await generateObject({
    model: openai('gpt-4'),
    schema: z.object({
      title: z.string(),
      summary: z.string(),
      tags: z.array(z.string()),
    }),
    prompt,
  });

  return result.object;
}
```

#### 工具调用

```javascript
/**
 * Vercel AI SDK 工具调用
 */
import { generateText, tool } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

// 定义工具
const weatherTool = tool({
  description: '获取指定城市的天气信息',
  parameters: z.object({
    city: z.string().describe('城市名称'),
    unit: z.enum(['celsius', 'fahrenheit']).optional(),
  }),
  execute: async ({ city, unit = 'celsius' }) => {
    // 调用天气 API
    const weather = await fetchWeather(city, unit);
    return weather;
  },
});

const searchTool = tool({
  description: '搜索互联网获取信息',
  parameters: z.object({
    query: z.string().describe('搜索关键词'),
  }),
  execute: async ({ query }) => {
    const results = await searchWeb(query);
    return results;
  },
});

// 使用工具
async function chatWithTools(prompt) {
  const result = await generateText({
    model: openai('gpt-4'),
    tools: {
      weather: weatherTool,
      search: searchTool,
    },
    prompt,
    maxToolRoundtrips: 5,  // 最大工具调用轮次
  });

  return result.text;
}
```

#### Next.js 集成

```javascript
/**
 * Next.js API Route (App Router)
 * app/api/chat/route.ts
 */
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4'),
    messages,
  });

  return result.toDataStreamResponse();
}
```

```tsx
/**
 * React 客户端组件
 * app/chat/page.tsx
 */
'use client';

import { useChat } from 'ai/react';

export default function Chat() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat',
  });

  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map((message) => (
          <div key={message.id} className={`message ${message.role}`}>
            {message.content}
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="输入消息..."
          disabled={isLoading}
        />
        <button type="submit" disabled={isLoading}>
          {isLoading ? '发送中...' : '发送'}
        </button>
      </form>
    </div>
  );
}
```

#### Vue 集成

```vue
<template>
  <div class="chat">
    <div class="messages">
      <div
        v-for="message in messages"
        :key="message.id"
        :class="['message', message.role]"
      >
        {{ message.content }}
      </div>
    </div>

    <form @submit.prevent="handleSubmit">
      <input
        v-model="input"
        placeholder="输入消息..."
        :disabled="isLoading"
      />
      <button type="submit" :disabled="isLoading">
        {{ isLoading ? '发送中...' : '发送' }}
      </button>
    </form>
  </div>
</template>

<script setup>
import { useChat } from '@ai-sdk/vue';

const { messages, input, handleSubmit, isLoading } = useChat({
  api: '/api/chat',
});
</script>
```

### 其他 Agent 框架

#### AutoGPT / AgentGPT

自主 Agent 框架，能够自动分解和执行复杂任务。

```javascript
/**
 * 类 AutoGPT 实现示例
 */
class AutonomousAgent {
  constructor(config) {
    this.llm = config.llm;
    this.tools = config.tools;
    this.goals = config.goals;
    this.memory = new AgentMemory();
  }

  async run() {
    while (true) {
      // 1. 思考当前状态
      const thought = await this.think();

      // 2. 检查是否完成目标
      if (thought.goalsCompleted) {
        return this.summarize();
      }

      // 3. 选择行动
      const action = await this.selectAction(thought);

      // 4. 执行行动
      const result = await this.executeAction(action);

      // 5. 更新记忆
      this.memory.add({ thought, action, result });

      // 6. 用户确认（可选）
      if (action.requiresConfirmation) {
        const approved = await this.getUserApproval(action);
        if (!approved) continue;
      }
    }
  }

  async think() {
    const context = this.memory.getContext();

    const response = await this.llm.call(`
      你是一个自主 Agent，正在完成以下目标：
      ${this.goals.join('\n')}

      当前状态：
      ${context}

      分析当前进度，决定下一步行动。
    `);

    return this.parseThought(response);
  }
}
```

#### CrewAI (多 Agent 协作)

```javascript
/**
 * 多 Agent 协作框架示例
 */
class CrewAI {
  constructor() {
    this.agents = new Map();
    this.tasks = [];
  }

  // 定义 Agent
  defineAgent(config) {
    const agent = {
      name: config.name,
      role: config.role,
      goal: config.goal,
      backstory: config.backstory,
      tools: config.tools || [],
      llm: config.llm,
    };

    this.agents.set(config.name, agent);
    return this;
  }

  // 定义任务
  defineTask(config) {
    this.tasks.push({
      description: config.description,
      expectedOutput: config.expectedOutput,
      agent: config.agent,
      dependencies: config.dependencies || [],
    });
    return this;
  }

  // 执行任务
  async kickoff() {
    const results = new Map();

    // 按依赖顺序执行任务
    const sortedTasks = this.topologicalSort(this.tasks);

    for (const task of sortedTasks) {
      const agent = this.agents.get(task.agent);

      // 获取依赖任务的结果
      const dependencyResults = task.dependencies.map(dep => results.get(dep));

      // 执行任务
      const result = await this.executeTask(agent, task, dependencyResults);
      results.set(task.description, result);
    }

    return results;
  }

  async executeTask(agent, task, dependencyResults) {
    const prompt = `
      你是 ${agent.name}，${agent.role}。
      目标：${agent.goal}
      背景：${agent.backstory}

      任务：${task.description}
      期望输出：${task.expectedOutput}

      依赖任务结果：
      ${dependencyResults.join('\n')}

      请完成这个任务。
    `;

    return await agent.llm.call(prompt);
  }

  topologicalSort(tasks) {
    // 拓扑排序实现
    return tasks;
  }
}

// 使用示例
const crew = new CrewAI();

crew
  .defineAgent({
    name: 'researcher',
    role: '研究员',
    goal: '收集和分析信息',
    backstory: '你是一位经验丰富的技术研究员',
    llm: new ChatOpenAI(),
  })
  .defineAgent({
    name: 'writer',
    role: '技术作家',
    goal: '撰写高质量的技术文章',
    backstory: '你是一位专业的技术博主',
    llm: new ChatOpenAI(),
  })
  .defineTask({
    description: '研究 React 19 的新特性',
    expectedOutput: '详细的特性列表和分析',
    agent: 'researcher',
  })
  .defineTask({
    description: '撰写技术博客文章',
    expectedOutput: '一篇完整的博客文章',
    agent: 'writer',
    dependencies: ['研究 React 19 的新特性'],
  });

const results = await crew.kickoff();
```

---

## 向量数据库

### 主流向量数据库对比

| 数据库 | 类型 | 特点 | 适用场景 |
|--------|------|------|----------|
| Pinecone | 云服务 | 托管服务、易用 | 快速开发 |
| Chroma | 本地/云 | 开源、轻量 | 本地开发 |
| Milvus | 自部署 | 高性能、可扩展 | 大规模生产 |
| Weaviate | 自部署 | 内置多模态 | 多模态搜索 |
| Qdrant | 自部署 | Rust 编写、高性能 | 高性能需求 |
| pgvector | PostgreSQL | SQL 兼容 | 已有 PG 基础设施 |

### Pinecone

```javascript
/**
 * Pinecone 使用示例
 */
import { Pinecone } from '@pinecone-database/pinecone';

const pinecone = new Pinecone({
  apiKey: process.env.PINECONE_API_KEY,
});

// 获取索引
const index = pinecone.index('my-index');

// 插入向量
async function upsertVectors(vectors) {
  await index.upsert(vectors.map((v, i) => ({
    id: `vec-${i}`,
    values: v.embedding,
    metadata: v.metadata,
  })));
}

// 查询向量
async function queryVectors(embedding, topK = 5) {
  const results = await index.query({
    vector: embedding,
    topK,
    includeMetadata: true,
  });

  return results.matches;
}

// 带过滤的查询
async function queryWithFilter(embedding, filter) {
  const results = await index.query({
    vector: embedding,
    topK: 5,
    filter: {
      category: { $eq: filter.category },
      date: { $gte: filter.startDate },
    },
    includeMetadata: true,
  });

  return results.matches;
}
```

### Chroma

```javascript
/**
 * Chroma 使用示例
 */
import { ChromaClient, OpenAIEmbeddingFunction } from 'chromadb';

const client = new ChromaClient();

// 创建嵌入函数
const embedder = new OpenAIEmbeddingFunction({
  openai_api_key: process.env.OPENAI_API_KEY,
});

// 创建或获取集合
const collection = await client.getOrCreateCollection({
  name: 'my-collection',
  embeddingFunction: embedder,
});

// 添加文档
async function addDocuments(documents) {
  await collection.add({
    ids: documents.map((_, i) => `doc-${i}`),
    documents: documents.map(d => d.text),
    metadatas: documents.map(d => d.metadata),
  });
}

// 查询文档
async function queryDocuments(query, nResults = 5) {
  const results = await collection.query({
    queryTexts: [query],
    nResults,
  });

  return results;
}
```

---

## 开发工具

### Token 计数

```javascript
/**
 * Token 计数工具
 */
import { encoding_for_model } from 'tiktoken';

class TokenCounter {
  constructor(model = 'gpt-4') {
    this.encoding = encoding_for_model(model);
  }

  count(text) {
    return this.encoding.encode(text).length;
  }

  truncate(text, maxTokens) {
    const tokens = this.encoding.encode(text);
    if (tokens.length <= maxTokens) {
      return text;
    }

    const truncated = tokens.slice(0, maxTokens);
    return this.encoding.decode(truncated);
  }

  estimateCost(inputTokens, outputTokens, model = 'gpt-4') {
    const pricing = {
      'gpt-4': { input: 0.03, output: 0.06 },
      'gpt-3.5-turbo': { input: 0.0015, output: 0.002 },
    };

    const p = pricing[model];
    return (inputTokens / 1000) * p.input + (outputTokens / 1000) * p.output;
  }
}
```

### 日志和监控

```javascript
/**
 * Agent 日志系统
 */
class AgentLogger {
  constructor(config = {}) {
    this.logs = [];
    this.callbacks = config.callbacks || [];
  }

  log(level, message, data = {}) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      data,
    };

    this.logs.push(entry);

    // 触发回调
    this.callbacks.forEach(cb => cb(entry));

    // 控制台输出
    console[level](
      `[${entry.timestamp}] [${level.toUpperCase()}] ${message}`,
      data
    );
  }

  info(message, data) {
    this.log('info', message, data);
  }

  error(message, data) {
    this.log('error', message, data);
  }

  // 记录 LLM 调用
  logLLMCall(input, output, metadata) {
    this.log('info', 'LLM Call', {
      input: input.substring(0, 100) + '...',
      output: output.substring(0, 100) + '...',
      tokens: metadata.tokens,
      latency: metadata.latency,
    });
  }

  // 记录工具调用
  logToolCall(tool, input, output) {
    this.log('info', `Tool Call: ${tool}`, { input, output });
  }

  // 导出日志
  export() {
    return JSON.stringify(this.logs, null, 2);
  }
}
```

---

## 框架选择指南

### 选择建议

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| 快速原型 | Vercel AI SDK | 简单易用，前端友好 |
| 复杂 Agent | LangChain.js | 功能完整，生态丰富 |
| Next.js 项目 | Vercel AI SDK | 原生集成 |
| 多 Agent 协作 | LangChain + 自定义 | 灵活性高 |
| 本地部署 | LangChain + Ollama | 隐私安全 |

### 技术选型决策树

```
是否需要复杂的 Agent 逻辑？
├── 是 → 使用 LangChain.js
│   ├── 需要 RAG？→ 使用 LangChain + 向量数据库
│   ├── 需要工具调用？→ 使用 LangChain Agent
│   └── 需要记忆？→ 使用 LangChain Memory
│
└── 否 → 使用 Vercel AI SDK
    ├── Next.js 项目？→ 使用 useChat hook
    ├── Vue 项目？→ 使用 @ai-sdk/vue
    └── 纯 Node.js？→ 使用 generateText/streamText
```

---

## 常见面试题

**1. LangChain 和 Vercel AI SDK 有什么区别？**

**答案要点：**
- LangChain：功能完整的 Agent 框架，支持复杂链式调用、记忆、RAG 等
- Vercel AI SDK：轻量级，专注流式 UI，与 Next.js 深度集成
- LangChain 适合复杂 Agent，Vercel AI SDK 适合快速开发

**2. 如何选择向量数据库？**

**答案要点：**
- 快速开发：Pinecone（托管服务）
- 本地开发：Chroma（轻量开源）
- 生产环境：Milvus、Qdrant（高性能）
- 已有 PostgreSQL：pgvector

**3. Function Calling 是如何工作的？**

**答案要点：**
- 定义工具 schema（名称、描述、参数）
- LLM 决定是否需要调用工具
- 返回工具名称和参数
- 执行工具，将结果返回给 LLM
- LLM 基于工具结果生成最终响应

# AI Agent 实战案例

## 概述

本文通过多个完整的实战案例，展示 AI Agent 在不同场景下的应用。每个案例都包含完整的代码实现和详细解析。

---

## 案例一：智能客服 Agent

### 项目概述

构建一个能够自动回答客户问题、查询订单、处理退款的智能客服 Agent。

```
用户: "我想查一下订单 12345 的状态"
Agent: [调用订单查询工具] 您的订单 12345 当前状态为"配送中"，预计明天送达。

用户: "我想退货"
Agent: [调用退货政策工具] 好的，根据退货政策，您的订单符合退货条件。
       [调用退货申请工具] 已为您创建退货申请，退货单号为 R67890。
```

### 完整实现

```javascript
/**
 * 智能客服 Agent 完整实现
 */
import { ChatOpenAI } from '@langchain/openai';
import { AgentExecutor, createOpenAIFunctionsAgent } from 'langchain/agents';
import { DynamicStructuredTool } from '@langchain/core/tools';
import { ChatPromptTemplate, MessagesPlaceholder } from '@langchain/core/prompts';
import { z } from 'zod';

// ==================== 工具定义 ====================

// 订单查询工具
const orderQueryTool = new DynamicStructuredTool({
  name: 'query_order',
  description: '查询订单状态和详细信息',
  schema: z.object({
    orderId: z.string().describe('订单ID'),
  }),
  func: async ({ orderId }) => {
    // 模拟数据库查询
    const orders = {
      '12345': {
        id: '12345',
        status: '配送中',
        items: ['iPhone 15 Pro', '手机壳'],
        amount: 8999,
        estimatedDelivery: '2024-03-15',
        trackingNumber: 'SF1234567890',
      },
      '67890': {
        id: '67890',
        status: '已签收',
        items: ['MacBook Air M3'],
        amount: 9999,
        deliveredAt: '2024-03-10',
      },
    };

    const order = orders[orderId];
    if (!order) {
      return JSON.stringify({ error: '订单不存在' });
    }

    return JSON.stringify(order);
  },
});

// 退货政策查询工具
const refundPolicyTool = new DynamicStructuredTool({
  name: 'check_refund_policy',
  description: '检查订单是否符合退货退款政策',
  schema: z.object({
    orderId: z.string().describe('订单ID'),
    reason: z.string().describe('退货原因'),
  }),
  func: async ({ orderId, reason }) => {
    // 模拟政策检查
    const policy = {
      eligible: true,
      reason: '订单在7天无理由退货期内',
      refundAmount: 8999,
      refundMethod: '原路返回',
      estimatedRefundTime: '3-5个工作日',
    };

    return JSON.stringify(policy);
  },
});

// 创建退货申请工具
const createRefundTool = new DynamicStructuredTool({
  name: 'create_refund',
  description: '创建退货退款申请',
  schema: z.object({
    orderId: z.string().describe('订单ID'),
    reason: z.string().describe('退货原因'),
    refundMethod: z.enum(['original', 'wallet']).describe('退款方式'),
  }),
  func: async ({ orderId, reason, refundMethod }) => {
    // 模拟创建退货申请
    const refundId = `R${Date.now()}`;

    return JSON.stringify({
      success: true,
      refundId,
      message: '退货申请已创建',
      nextSteps: [
        '等待客服审核（1-2个工作日）',
        '寄回商品',
        '收到商品后处理退款',
      ],
    });
  },
});

// 物流查询工具
const trackingTool = new DynamicStructuredTool({
  name: 'track_shipping',
  description: '查询物流配送信息',
  schema: z.object({
    trackingNumber: z.string().describe('物流单号'),
  }),
  func: async ({ trackingNumber }) => {
    const tracking = {
      carrier: '顺丰速运',
      status: '派送中',
      history: [
        { time: '2024-03-14 08:00', event: '快递员正在派送' },
        { time: '2024-03-13 20:00', event: '到达北京转运中心' },
        { time: '2024-03-13 10:00', event: '深圳发出' },
      ],
    };

    return JSON.stringify(tracking);
  },
});

// 人工客服转接工具
const transferToHumanTool = new DynamicStructuredTool({
  name: 'transfer_to_human',
  description: '将对话转接给人工客服，用于处理复杂问题或用户明确要求人工服务',
  schema: z.object({
    reason: z.string().describe('转接原因'),
    summary: z.string().describe('对话摘要'),
  }),
  func: async ({ reason, summary }) => {
    return JSON.stringify({
      success: true,
      message: '正在为您转接人工客服，预计等待时间 2 分钟',
      ticketId: `T${Date.now()}`,
    });
  },
});

// ==================== Agent 创建 ====================

async function createCustomerServiceAgent() {
  const llm = new ChatOpenAI({
    modelName: 'gpt-4',
    temperature: 0,
  });

  const tools = [
    orderQueryTool,
    refundPolicyTool,
    createRefundTool,
    trackingTool,
    transferToHumanTool,
  ];

  const prompt = ChatPromptTemplate.fromMessages([
    [
      'system',
      `你是一个专业、友好的智能客服助手。

职责：
1. 帮助用户查询订单状态和物流信息
2. 处理退货退款请求
3. 解答常见问题
4. 必要时转接人工客服

规则：
1. 始终保持礼貌和专业
2. 在执行敏感操作（如退款）前，先确认用户意图
3. 如果无法解决问题，主动提供人工服务
4. 保护用户隐私，不透露敏感信息
5. 使用中文回复

当前时间：{current_time}`,
    ],
    new MessagesPlaceholder('chat_history'),
    ['human', '{input}'],
    new MessagesPlaceholder('agent_scratchpad'),
  ]);

  const agent = await createOpenAIFunctionsAgent({
    llm,
    tools,
    prompt,
  });

  return new AgentExecutor({
    agent,
    tools,
    verbose: true,
  });
}

// ==================== 对话管理 ====================

class CustomerServiceChat {
  constructor() {
    this.agent = null;
    this.chatHistory = [];
  }

  async initialize() {
    this.agent = await createCustomerServiceAgent();
  }

  async chat(userMessage) {
    const result = await this.agent.invoke({
      input: userMessage,
      chat_history: this.chatHistory,
      current_time: new Date().toLocaleString('zh-CN'),
    });

    // 更新对话历史
    this.chatHistory.push(
      { role: 'user', content: userMessage },
      { role: 'assistant', content: result.output }
    );

    return result.output;
  }
}

// ==================== 使用示例 ====================

async function main() {
  const chat = new CustomerServiceChat();
  await chat.initialize();

  // 模拟对话
  console.log('用户: 你好，我想查一下订单 12345 的状态');
  console.log('客服:', await chat.chat('你好，我想查一下订单 12345 的状态'));

  console.log('\n用户: 物流到哪了？');
  console.log('客服:', await chat.chat('物流到哪了？'));

  console.log('\n用户: 我想退货');
  console.log('客服:', await chat.chat('我想退货'));
}

main();
```

### Vue 3 前端实现

```vue
<template>
  <div class="customer-service">
    <div class="chat-header">
      <h3>智能客服</h3>
      <span class="status">在线</span>
    </div>

    <div class="messages" ref="messagesRef">
      <div
        v-for="message in messages"
        :key="message.id"
        :class="['message', message.role]"
      >
        <div class="avatar">
          {{ message.role === 'user' ? '我' : '客服' }}
        </div>
        <div class="content">
          <div class="text">{{ message.content }}</div>
          <div class="time">{{ message.time }}</div>
        </div>
      </div>

      <div v-if="isLoading" class="message assistant">
        <div class="avatar">客服</div>
        <div class="content">
          <div class="typing">正在输入...</div>
        </div>
      </div>
    </div>

    <div class="quick-actions">
      <button
        v-for="action in quickActions"
        :key="action"
        @click="sendQuickAction(action)"
      >
        {{ action }}
      </button>
    </div>

    <div class="input-area">
      <input
        v-model="input"
        @keyup.enter="sendMessage"
        placeholder="请输入您的问题..."
        :disabled="isLoading"
      />
      <button @click="sendMessage" :disabled="isLoading || !input.trim()">
        发送
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue';

const messages = ref([
  {
    id: 1,
    role: 'assistant',
    content: '您好！我是智能客服，请问有什么可以帮您？',
    time: formatTime(new Date()),
  },
]);

const input = ref('');
const isLoading = ref(false);
const messagesRef = ref(null);

const quickActions = [
  '查询订单',
  '物流查询',
  '申请退款',
  '联系人工客服',
];

function formatTime(date) {
  return date.toLocaleTimeString('zh-CN', {
    hour: '2-digit',
    minute: '2-digit',
  });
}

async function sendMessage() {
  if (!input.value.trim() || isLoading.value) return;

  const userMessage = {
    id: Date.now(),
    role: 'user',
    content: input.value,
    time: formatTime(new Date()),
  };

  messages.value.push(userMessage);
  const userInput = input.value;
  input.value = '';
  isLoading.value = true;

  await scrollToBottom();

  try {
    const response = await fetch('/api/customer-service', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        message: userInput,
        history: messages.value.slice(0, -1),
      }),
    });

    const data = await response.json();

    messages.value.push({
      id: Date.now() + 1,
      role: 'assistant',
      content: data.response,
      time: formatTime(new Date()),
    });
  } catch (error) {
    messages.value.push({
      id: Date.now() + 1,
      role: 'assistant',
      content: '抱歉，系统出现问题，请稍后再试或联系人工客服。',
      time: formatTime(new Date()),
    });
  } finally {
    isLoading.value = false;
    await scrollToBottom();
  }
}

function sendQuickAction(action) {
  input.value = action;
  sendMessage();
}

async function scrollToBottom() {
  await nextTick();
  if (messagesRef.value) {
    messagesRef.value.scrollTop = messagesRef.value.scrollHeight;
  }
}
</script>

<style scoped>
.customer-service {
  display: flex;
  flex-direction: column;
  height: 600px;
  max-width: 400px;
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  overflow: hidden;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  background: #1890ff;
  color: white;
}

.status {
  font-size: 12px;
  padding: 2px 8px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 10px;
}

.messages {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  background: #f5f5f5;
}

.message {
  display: flex;
  margin-bottom: 16px;
}

.message.user {
  flex-direction: row-reverse;
}

.avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #1890ff;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  flex-shrink: 0;
}

.message.user .avatar {
  background: #52c41a;
}

.content {
  max-width: 70%;
  margin: 0 8px;
}

.text {
  padding: 10px 14px;
  background: white;
  border-radius: 12px;
  line-height: 1.5;
}

.message.user .text {
  background: #1890ff;
  color: white;
}

.time {
  font-size: 11px;
  color: #999;
  margin-top: 4px;
  text-align: right;
}

.typing {
  padding: 10px 14px;
  background: white;
  border-radius: 12px;
  color: #999;
}

.quick-actions {
  display: flex;
  gap: 8px;
  padding: 8px 16px;
  overflow-x: auto;
  background: white;
  border-top: 1px solid #e0e0e0;
}

.quick-actions button {
  flex-shrink: 0;
  padding: 6px 12px;
  border: 1px solid #1890ff;
  border-radius: 16px;
  background: white;
  color: #1890ff;
  cursor: pointer;
  font-size: 13px;
}

.quick-actions button:hover {
  background: #e6f7ff;
}

.input-area {
  display: flex;
  padding: 12px 16px;
  background: white;
  border-top: 1px solid #e0e0e0;
}

.input-area input {
  flex: 1;
  padding: 10px 14px;
  border: 1px solid #d9d9d9;
  border-radius: 20px;
  outline: none;
}

.input-area input:focus {
  border-color: #1890ff;
}

.input-area button {
  margin-left: 8px;
  padding: 10px 20px;
  background: #1890ff;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
}

.input-area button:disabled {
  background: #d9d9d9;
  cursor: not-allowed;
}
</style>
```

---

## 案例二：文档问答 Agent (RAG)

### 项目概述

构建一个能够回答公司内部文档问题的 RAG Agent，支持上传文档、智能检索、准确回答。

### 完整实现

```javascript
/**
 * 文档问答 RAG Agent
 */
import { ChatOpenAI, OpenAIEmbeddings } from '@langchain/openai';
import { MemoryVectorStore } from 'langchain/vectorstores/memory';
import { RecursiveCharacterTextSplitter } from 'langchain/text_splitter';
import { createRetrievalChain } from 'langchain/chains/retrieval';
import { createStuffDocumentsChain } from 'langchain/chains/combine_documents';
import { ChatPromptTemplate, MessagesPlaceholder } from '@langchain/core/prompts';
import { Document } from '@langchain/core/documents';
import { PDFLoader } from 'langchain/document_loaders/fs/pdf';
import { TextLoader } from 'langchain/document_loaders/fs/text';

// ==================== 文档处理 ====================

class DocumentProcessor {
  constructor() {
    this.textSplitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200,
      separators: ['\n\n', '\n', '。', '！', '？', '；', ' ', ''],
    });
  }

  // 加载不同格式的文档
  async loadDocument(filePath) {
    const extension = filePath.split('.').pop().toLowerCase();

    let loader;
    switch (extension) {
      case 'pdf':
        loader = new PDFLoader(filePath);
        break;
      case 'txt':
      case 'md':
        loader = new TextLoader(filePath);
        break;
      default:
        throw new Error(`不支持的文件格式: ${extension}`);
    }

    return await loader.load();
  }

  // 分割文档
  async splitDocuments(documents) {
    return await this.textSplitter.splitDocuments(documents);
  }

  // 从文本创建文档
  createDocumentsFromText(texts, metadata = {}) {
    return texts.map(
      (text, i) =>
        new Document({
          pageContent: text,
          metadata: { ...metadata, chunk: i },
        })
    );
  }
}

// ==================== RAG 系统 ====================

class RAGDocumentAgent {
  constructor() {
    this.llm = new ChatOpenAI({
      modelName: 'gpt-4',
      temperature: 0,
    });

    this.embeddings = new OpenAIEmbeddings();
    this.vectorStore = null;
    this.retriever = null;
    this.chain = null;
    this.processor = new DocumentProcessor();
    this.chatHistory = [];
  }

  // 初始化向量存储
  async initialize(documents) {
    // 分割文档
    const splits = await this.processor.splitDocuments(documents);

    // 创建向量存储
    this.vectorStore = await MemoryVectorStore.fromDocuments(
      splits,
      this.embeddings
    );

    // 创建检索器
    this.retriever = this.vectorStore.asRetriever({
      k: 4,
      searchType: 'similarity',
    });

    // 创建问答链
    await this.createChain();

    console.log(`已加载 ${splits.length} 个文档片段`);
  }

  // 添加文档
  async addDocuments(documents) {
    const splits = await this.processor.splitDocuments(documents);
    await this.vectorStore.addDocuments(splits);
    console.log(`新增 ${splits.length} 个文档片段`);
  }

  // 创建问答链
  async createChain() {
    const systemPrompt = `你是一个专业的文档问答助手。基于提供的上下文回答用户问题。

规则：
1. 只基于提供的上下文回答，不要编造信息
2. 如果上下文中没有相关信息，明确说明"文档中没有相关信息"
3. 引用来源，说明信息来自哪个文档
4. 保持回答简洁、准确
5. 如果问题不清楚，请求用户澄清

上下文：
{context}`;

    const prompt = ChatPromptTemplate.fromMessages([
      ['system', systemPrompt],
      new MessagesPlaceholder('chat_history'),
      ['human', '{input}'],
    ]);

    const documentChain = await createStuffDocumentsChain({
      llm: this.llm,
      prompt,
    });

    this.chain = await createRetrievalChain({
      combineDocsChain: documentChain,
      retriever: this.retriever,
    });
  }

  // 问答
  async query(question) {
    if (!this.chain) {
      throw new Error('请先初始化文档');
    }

    const result = await this.chain.invoke({
      input: question,
      chat_history: this.chatHistory,
    });

    // 更新对话历史
    this.chatHistory.push(
      { role: 'user', content: question },
      { role: 'assistant', content: result.answer }
    );

    // 保持历史长度
    if (this.chatHistory.length > 10) {
      this.chatHistory = this.chatHistory.slice(-10);
    }

    return {
      answer: result.answer,
      sources: result.context.map((doc) => ({
        content: doc.pageContent.substring(0, 200) + '...',
        metadata: doc.metadata,
      })),
    };
  }

  // 相似度搜索
  async similaritySearch(query, k = 4) {
    return await this.vectorStore.similaritySearch(query, k);
  }

  // 清空历史
  clearHistory() {
    this.chatHistory = [];
  }
}

// ==================== 高级 RAG 功能 ====================

class AdvancedRAGAgent extends RAGDocumentAgent {
  constructor() {
    super();
    this.queryRewriter = null;
  }

  // 查询重写（提高检索效果）
  async rewriteQuery(query) {
    const response = await this.llm.invoke([
      {
        role: 'system',
        content: `你是一个查询优化专家。将用户的问题改写为更适合文档检索的形式。
输出3个不同角度的查询，用换行分隔。`,
      },
      {
        role: 'user',
        content: query,
      },
    ]);

    return response.content.split('\n').filter((q) => q.trim());
  }

  // 多查询检索
  async multiQueryRetrieve(query, k = 4) {
    const queries = await this.rewriteQuery(query);
    queries.unshift(query); // 添加原始查询

    const allDocs = [];
    const seenIds = new Set();

    for (const q of queries) {
      const docs = await this.vectorStore.similaritySearch(q, k);
      for (const doc of docs) {
        const docId = doc.pageContent.substring(0, 100);
        if (!seenIds.has(docId)) {
          seenIds.add(docId);
          allDocs.push(doc);
        }
      }
    }

    return allDocs.slice(0, k * 2);
  }

  // 带重排序的问答
  async queryWithReranking(question) {
    // 1. 多查询检索
    const docs = await this.multiQueryRetrieve(question, 6);

    // 2. 重排序（使用 LLM 评估相关性）
    const rankedDocs = await this.rerankDocuments(question, docs);

    // 3. 生成答案
    const context = rankedDocs
      .slice(0, 4)
      .map((d) => d.pageContent)
      .join('\n\n---\n\n');

    const response = await this.llm.invoke([
      {
        role: 'system',
        content: `基于以下上下文回答问题。如果上下文中没有相关信息，说明"文档中没有相关信息"。

上下文：
${context}`,
      },
      {
        role: 'user',
        content: question,
      },
    ]);

    return {
      answer: response.content,
      sources: rankedDocs.slice(0, 4).map((d) => ({
        content: d.pageContent.substring(0, 200) + '...',
        score: d.score,
        metadata: d.metadata,
      })),
    };
  }

  // 文档重排序
  async rerankDocuments(query, docs) {
    const scoredDocs = await Promise.all(
      docs.map(async (doc) => {
        const response = await this.llm.invoke([
          {
            role: 'system',
            content: `评估以下文档与查询的相关性。只输出一个 0-10 的分数。`,
          },
          {
            role: 'user',
            content: `查询：${query}\n\n文档：${doc.pageContent.substring(0, 500)}`,
          },
        ]);

        const score = parseFloat(response.content) || 0;
        return { ...doc, score };
      })
    );

    return scoredDocs.sort((a, b) => b.score - a.score);
  }
}

// ==================== 使用示例 ====================

async function main() {
  const agent = new AdvancedRAGAgent();

  // 模拟文档
  const documents = [
    new Document({
      pageContent: `
# 公司休假政策

## 年假
- 入职满一年：5天年假
- 入职满三年：10天年假
- 入职满五年：15天年假

## 病假
- 每年最多15天带薪病假
- 需提供医院证明

## 请假流程
1. 在 OA 系统提交请假申请
2. 直属上级审批
3. HR 备案
      `,
      metadata: { source: '员工手册.md', section: '休假政策' },
    }),
    new Document({
      pageContent: `
# 报销规定

## 差旅报销
- 飞机：经济舱标准
- 火车：二等座标准
- 酒店：每晚不超过500元

## 餐饮报销
- 工作日加班餐：每人每餐不超过50元
- 出差餐补：每天150元

## 报销流程
1. 收集发票和行程单
2. 在费用系统填写报销单
3. 上传发票照片
4. 提交审批
      `,
      metadata: { source: '财务制度.md', section: '报销规定' },
    }),
  ];

  await agent.initialize(documents);

  // 测试问答
  console.log('问：入职两年有几天年假？');
  const result1 = await agent.query('入职两年有几天年假？');
  console.log('答：', result1.answer);
  console.log('来源：', result1.sources);

  console.log('\n问：出差住酒店的标准是多少？');
  const result2 = await agent.query('出差住酒店的标准是多少？');
  console.log('答：', result2.answer);
}

main();
```

---

## 案例三：代码助手 Agent

### 项目概述

构建一个能够理解代码、回答编程问题、生成代码的编程助手 Agent。

### 完整实现

```javascript
/**
 * 代码助手 Agent
 */
import { ChatOpenAI } from '@langchain/openai';
import { DynamicStructuredTool } from '@langchain/core/tools';
import { AgentExecutor, createOpenAIFunctionsAgent } from 'langchain/agents';
import { ChatPromptTemplate, MessagesPlaceholder } from '@langchain/core/prompts';
import { z } from 'zod';
import * as fs from 'fs/promises';
import * as path from 'path';

// ==================== 工具定义 ====================

// 读取文件工具
const readFileTool = new DynamicStructuredTool({
  name: 'read_file',
  description: '读取指定路径的文件内容',
  schema: z.object({
    filePath: z.string().describe('文件路径'),
  }),
  func: async ({ filePath }) => {
    try {
      const content = await fs.readFile(filePath, 'utf-8');
      return content;
    } catch (error) {
      return `读取文件失败: ${error.message}`;
    }
  },
});

// 写入文件工具
const writeFileTool = new DynamicStructuredTool({
  name: 'write_file',
  description: '将内容写入指定文件',
  schema: z.object({
    filePath: z.string().describe('文件路径'),
    content: z.string().describe('文件内容'),
  }),
  func: async ({ filePath, content }) => {
    try {
      await fs.writeFile(filePath, content, 'utf-8');
      return `文件已成功写入: ${filePath}`;
    } catch (error) {
      return `写入文件失败: ${error.message}`;
    }
  },
});

// 列出目录工具
const listDirectoryTool = new DynamicStructuredTool({
  name: 'list_directory',
  description: '列出目录中的文件和子目录',
  schema: z.object({
    dirPath: z.string().describe('目录路径'),
  }),
  func: async ({ dirPath }) => {
    try {
      const entries = await fs.readdir(dirPath, { withFileTypes: true });
      const result = entries.map((entry) => ({
        name: entry.name,
        type: entry.isDirectory() ? 'directory' : 'file',
      }));
      return JSON.stringify(result, null, 2);
    } catch (error) {
      return `读取目录失败: ${error.message}`;
    }
  },
});

// 搜索代码工具
const searchCodeTool = new DynamicStructuredTool({
  name: 'search_code',
  description: '在代码库中搜索指定的代码模式',
  schema: z.object({
    pattern: z.string().describe('搜索模式（支持正则表达式）'),
    directory: z.string().describe('搜索目录'),
    fileExtension: z.string().optional().describe('文件扩展名过滤'),
  }),
  func: async ({ pattern, directory, fileExtension }) => {
    const results = [];
    const regex = new RegExp(pattern, 'gi');

    async function searchDir(dir) {
      const entries = await fs.readdir(dir, { withFileTypes: true });

      for (const entry of entries) {
        const fullPath = path.join(dir, entry.name);

        if (entry.isDirectory() && !entry.name.startsWith('.') && entry.name !== 'node_modules') {
          await searchDir(fullPath);
        } else if (entry.isFile()) {
          if (fileExtension && !entry.name.endsWith(fileExtension)) {
            continue;
          }

          try {
            const content = await fs.readFile(fullPath, 'utf-8');
            const lines = content.split('\n');

            lines.forEach((line, index) => {
              if (regex.test(line)) {
                results.push({
                  file: fullPath,
                  line: index + 1,
                  content: line.trim(),
                });
              }
            });
          } catch (e) {
            // 忽略无法读取的文件
          }
        }
      }
    }

    await searchDir(directory);

    if (results.length === 0) {
      return '未找到匹配的代码';
    }

    return JSON.stringify(results.slice(0, 20), null, 2);
  },
});

// 执行代码工具（沙箱环境）
const executeCodeTool = new DynamicStructuredTool({
  name: 'execute_code',
  description: '在沙箱环境中执行 JavaScript 代码',
  schema: z.object({
    code: z.string().describe('要执行的 JavaScript 代码'),
  }),
  func: async ({ code }) => {
    try {
      // 注意：实际生产环境需要使用真正的沙箱
      const AsyncFunction = Object.getPrototypeOf(async function () {}).constructor;
      const fn = new AsyncFunction(code);
      const result = await fn();
      return `执行结果: ${JSON.stringify(result, null, 2)}`;
    } catch (error) {
      return `执行错误: ${error.message}`;
    }
  },
});

// 代码分析工具
const analyzeCodeTool = new DynamicStructuredTool({
  name: 'analyze_code',
  description: '分析代码的复杂度、潜在问题和改进建议',
  schema: z.object({
    code: z.string().describe('要分析的代码'),
    language: z.string().describe('编程语言'),
  }),
  func: async ({ code, language }) => {
    // 简单的代码分析
    const analysis = {
      lineCount: code.split('\n').length,
      hasConsoleLog: /console\.log/.test(code),
      hasComments: /\/\/|\/\*/.test(code),
      hasTryCatch: /try\s*{/.test(code),
      functionCount: (code.match(/function\s+\w+|const\s+\w+\s*=\s*(\([^)]*\)|async\s*\([^)]*\))\s*=>/g) || [])
        .length,
    };

    const suggestions = [];
    if (analysis.hasConsoleLog) {
      suggestions.push('建议移除 console.log 语句');
    }
    if (!analysis.hasComments && analysis.lineCount > 20) {
      suggestions.push('建议添加代码注释');
    }
    if (!analysis.hasTryCatch && /await|fetch|axios/.test(code)) {
      suggestions.push('建议添加错误处理');
    }

    return JSON.stringify({ analysis, suggestions }, null, 2);
  },
});

// ==================== Agent 创建 ====================

async function createCodeAssistantAgent() {
  const llm = new ChatOpenAI({
    modelName: 'gpt-4',
    temperature: 0,
  });

  const tools = [
    readFileTool,
    writeFileTool,
    listDirectoryTool,
    searchCodeTool,
    executeCodeTool,
    analyzeCodeTool,
  ];

  const prompt = ChatPromptTemplate.fromMessages([
    [
      'system',
      `你是一个专业的编程助手，精通多种编程语言和最佳实践。

能力：
1. 阅读和理解代码
2. 编写高质量代码
3. 分析代码问题
4. 提供优化建议
5. 解释编程概念

规则：
1. 代码要清晰、可维护
2. 遵循最佳实践和设计模式
3. 添加必要的注释和文档
4. 考虑边界情况和错误处理
5. 提供代码示例时要完整可运行

工作目录: {working_directory}`,
    ],
    new MessagesPlaceholder('chat_history'),
    ['human', '{input}'],
    new MessagesPlaceholder('agent_scratchpad'),
  ]);

  const agent = await createOpenAIFunctionsAgent({
    llm,
    tools,
    prompt,
  });

  return new AgentExecutor({
    agent,
    tools,
    verbose: true,
    maxIterations: 10,
  });
}

// ==================== 代码助手类 ====================

class CodeAssistant {
  constructor(workingDirectory = process.cwd()) {
    this.workingDirectory = workingDirectory;
    this.agent = null;
    this.chatHistory = [];
  }

  async initialize() {
    this.agent = await createCodeAssistantAgent();
  }

  async chat(input) {
    const result = await this.agent.invoke({
      input,
      chat_history: this.chatHistory,
      working_directory: this.workingDirectory,
    });

    this.chatHistory.push(
      { role: 'user', content: input },
      { role: 'assistant', content: result.output }
    );

    return result.output;
  }

  // 代码生成
  async generateCode(description, language = 'javascript') {
    return await this.chat(
      `请用 ${language} 实现以下功能：\n${description}\n\n要求：
1. 代码要完整可运行
2. 添加必要的注释
3. 考虑错误处理
4. 遵循最佳实践`
    );
  }

  // 代码解释
  async explainCode(code) {
    return await this.chat(
      `请详细解释以下代码：\n\`\`\`\n${code}\n\`\`\`\n\n包括：
1. 整体功能
2. 关键逻辑
3. 使用的技术/模式
4. 潜在问题`
    );
  }

  // 代码优化
  async optimizeCode(code) {
    return await this.chat(
      `请分析并优化以下代码：\n\`\`\`\n${code}\n\`\`\`\n\n从以下角度：
1. 性能优化
2. 可读性改进
3. 最佳实践
4. 安全性考虑
5. 给出优化后的完整代码`
    );
  }

  // 代码审查
  async reviewCode(code) {
    return await this.chat(
      `请对以下代码进行代码审查：\n\`\`\`\n${code}\n\`\`\`\n\n检查：
1. 代码质量
2. 潜在 bug
3. 安全漏洞
4. 性能问题
5. 改进建议`
    );
  }

  // 生成单元测试
  async generateTests(code) {
    return await this.chat(
      `请为以下代码生成单元测试：\n\`\`\`\n${code}\n\`\`\`\n\n要求：
1. 使用 Jest 框架
2. 覆盖正常情况
3. 覆盖边界情况
4. 覆盖错误情况
5. 每个测试要有清晰的描述`
    );
  }
}

// ==================== 使用示例 ====================

async function main() {
  const assistant = new CodeAssistant();
  await assistant.initialize();

  // 代码生成
  console.log('=== 代码生成 ===');
  const generated = await assistant.generateCode(
    '实现一个带缓存的 API 请求函数，支持过期时间设置'
  );
  console.log(generated);

  // 代码解释
  console.log('\n=== 代码解释 ===');
  const explanation = await assistant.explainCode(`
function debounce(fn, delay) {
  let timer = null;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
  `);
  console.log(explanation);

  // 代码优化
  console.log('\n=== 代码优化 ===');
  const optimized = await assistant.optimizeCode(`
function getUser(id) {
  var user = null;
  for (var i = 0; i < users.length; i++) {
    if (users[i].id == id) {
      user = users[i];
      break;
    }
  }
  return user;
}
  `);
  console.log(optimized);
}

main();
```

---

## 案例四：自动化工作流 Agent

### 项目概述

构建一个能够自动执行日常任务的工作流 Agent，如定时汇报、数据收集、消息通知等。

### 完整实现

```javascript
/**
 * 自动化工作流 Agent
 */
import { ChatOpenAI } from '@langchain/openai';
import { DynamicStructuredTool } from '@langchain/core/tools';
import { z } from 'zod';
import cron from 'node-cron';

// ==================== 工具定义 ====================

// 发送邮件工具
const sendEmailTool = new DynamicStructuredTool({
  name: 'send_email',
  description: '发送电子邮件',
  schema: z.object({
    to: z.string().describe('收件人邮箱'),
    subject: z.string().describe('邮件主题'),
    body: z.string().describe('邮件内容'),
  }),
  func: async ({ to, subject, body }) => {
    // 模拟发送邮件
    console.log(`发送邮件到 ${to}: ${subject}`);
    return JSON.stringify({
      success: true,
      messageId: `MSG-${Date.now()}`,
    });
  },
});

// 发送 Slack 消息工具
const sendSlackTool = new DynamicStructuredTool({
  name: 'send_slack',
  description: '发送 Slack 消息',
  schema: z.object({
    channel: z.string().describe('频道名称'),
    message: z.string().describe('消息内容'),
  }),
  func: async ({ channel, message }) => {
    console.log(`发送 Slack 消息到 #${channel}: ${message}`);
    return JSON.stringify({ success: true });
  },
});

// 获取数据工具
const fetchDataTool = new DynamicStructuredTool({
  name: 'fetch_data',
  description: '从 API 获取数据',
  schema: z.object({
    source: z.enum(['sales', 'users', 'errors', 'performance']).describe('数据源'),
    dateRange: z.string().describe('日期范围，如 "today", "week", "month"'),
  }),
  func: async ({ source, dateRange }) => {
    // 模拟数据获取
    const mockData = {
      sales: {
        today: { total: 15680, orders: 42, avgOrder: 373 },
        week: { total: 98500, orders: 256, avgOrder: 385 },
      },
      users: {
        today: { newUsers: 128, activeUsers: 1560 },
        week: { newUsers: 856, activeUsers: 8920 },
      },
      errors: {
        today: { count: 23, critical: 2 },
        week: { count: 156, critical: 8 },
      },
      performance: {
        today: { avgResponseTime: 245, p99: 890 },
        week: { avgResponseTime: 238, p99: 920 },
      },
    };

    return JSON.stringify(mockData[source]?.[dateRange] || {});
  },
});

// 生成报告工具
const generateReportTool = new DynamicStructuredTool({
  name: 'generate_report',
  description: '生成指定类型的报告',
  schema: z.object({
    type: z.enum(['daily', 'weekly', 'monthly']).describe('报告类型'),
    data: z.string().describe('报告数据（JSON 格式）'),
  }),
  func: async ({ type, data }) => {
    const reportData = JSON.parse(data);
    const report = {
      generatedAt: new Date().toISOString(),
      type,
      data: reportData,
    };
    return JSON.stringify(report, null, 2);
  },
});

// 创建日历事件工具
const createCalendarEventTool = new DynamicStructuredTool({
  name: 'create_calendar_event',
  description: '创建日历事件',
  schema: z.object({
    title: z.string().describe('事件标题'),
    startTime: z.string().describe('开始时间'),
    endTime: z.string().describe('结束时间'),
    attendees: z.array(z.string()).describe('参与者邮箱列表'),
  }),
  func: async ({ title, startTime, endTime, attendees }) => {
    console.log(`创建日历事件: ${title}`);
    return JSON.stringify({
      success: true,
      eventId: `EVT-${Date.now()}`,
    });
  },
});

// ==================== 工作流 Agent ====================

class WorkflowAgent {
  constructor() {
    this.llm = new ChatOpenAI({
      modelName: 'gpt-4',
      temperature: 0,
    });
    this.tools = [
      sendEmailTool,
      sendSlackTool,
      fetchDataTool,
      generateReportTool,
      createCalendarEventTool,
    ];
    this.scheduledTasks = new Map();
  }

  // 执行工作流
  async executeWorkflow(workflow) {
    const results = [];

    for (const step of workflow.steps) {
      console.log(`执行步骤: ${step.name}`);

      try {
        const result = await this.executeStep(step);
        results.push({ step: step.name, success: true, result });

        // 如果有条件判断
        if (step.condition) {
          const shouldContinue = await this.evaluateCondition(step.condition, result);
          if (!shouldContinue) {
            console.log(`条件不满足，跳过后续步骤`);
            break;
          }
        }
      } catch (error) {
        results.push({ step: step.name, success: false, error: error.message });

        if (step.onError === 'abort') {
          throw error;
        }
      }
    }

    return results;
  }

  // 执行单个步骤
  async executeStep(step) {
    const tool = this.tools.find((t) => t.name === step.tool);

    if (!tool) {
      throw new Error(`工具不存在: ${step.tool}`);
    }

    return await tool.func(step.params);
  }

  // 评估条件
  async evaluateCondition(condition, data) {
    const response = await this.llm.invoke([
      {
        role: 'system',
        content: '评估条件是否满足，只回答 true 或 false',
      },
      {
        role: 'user',
        content: `条件: ${condition}\n数据: ${data}`,
      },
    ]);

    return response.content.toLowerCase().includes('true');
  }

  // 定时任务
  scheduleWorkflow(name, cronExpression, workflow) {
    const task = cron.schedule(cronExpression, async () => {
      console.log(`执行定时任务: ${name}`);
      try {
        await this.executeWorkflow(workflow);
      } catch (error) {
        console.error(`任务执行失败: ${error.message}`);
      }
    });

    this.scheduledTasks.set(name, task);
    console.log(`已调度任务: ${name} (${cronExpression})`);
  }

  // 取消定时任务
  cancelScheduledWorkflow(name) {
    const task = this.scheduledTasks.get(name);
    if (task) {
      task.stop();
      this.scheduledTasks.delete(name);
      console.log(`已取消任务: ${name}`);
    }
  }
}

// ==================== 预定义工作流 ====================

// 每日报告工作流
const dailyReportWorkflow = {
  name: '每日数据报告',
  steps: [
    {
      name: '获取销售数据',
      tool: 'fetch_data',
      params: { source: 'sales', dateRange: 'today' },
    },
    {
      name: '获取用户数据',
      tool: 'fetch_data',
      params: { source: 'users', dateRange: 'today' },
    },
    {
      name: '获取错误数据',
      tool: 'fetch_data',
      params: { source: 'errors', dateRange: 'today' },
    },
    {
      name: '生成报告',
      tool: 'generate_report',
      params: {
        type: 'daily',
        data: '{{previousResults}}', // 使用前面步骤的结果
      },
    },
    {
      name: '发送 Slack 通知',
      tool: 'send_slack',
      params: {
        channel: 'daily-reports',
        message: '每日数据报告已生成，请查看',
      },
    },
    {
      name: '发送邮件',
      tool: 'send_email',
      params: {
        to: 'team@example.com',
        subject: '每日数据报告',
        body: '{{report}}',
      },
    },
  ],
};

// 错误监控工作流
const errorMonitorWorkflow = {
  name: '错误监控告警',
  steps: [
    {
      name: '检查错误',
      tool: 'fetch_data',
      params: { source: 'errors', dateRange: 'today' },
      condition: 'critical > 0', // 只有存在严重错误时继续
    },
    {
      name: '发送告警',
      tool: 'send_slack',
      params: {
        channel: 'alerts',
        message: '检测到严重错误，请立即处理！',
      },
    },
    {
      name: '发送邮件告警',
      tool: 'send_email',
      params: {
        to: 'oncall@example.com',
        subject: '严重错误告警',
        body: '系统检测到严重错误，请立即查看。',
      },
    },
  ],
};

// ==================== 使用示例 ====================

async function main() {
  const agent = new WorkflowAgent();

  // 立即执行工作流
  console.log('=== 执行每日报告工作流 ===');
  const results = await agent.executeWorkflow(dailyReportWorkflow);
  console.log('执行结果:', JSON.stringify(results, null, 2));

  // 调度定时任务
  // 每天早上 9 点执行每日报告
  agent.scheduleWorkflow('daily-report', '0 9 * * *', dailyReportWorkflow);

  // 每 5 分钟检查错误
  agent.scheduleWorkflow('error-monitor', '*/5 * * * *', errorMonitorWorkflow);
}

main();
```

---

## 案例总结

### 案例对比

| 案例 | 核心技术 | 复杂度 | 适用场景 |
|------|---------|--------|----------|
| 智能客服 | Agent + Tools | 中 | 客户服务自动化 |
| 文档问答 | RAG | 中高 | 知识库问答 |
| 代码助手 | Agent + Tools | 高 | 开发效率提升 |
| 工作流 | Agent + 调度 | 中 | 业务自动化 |

### 关键技术点

1. **工具设计**：定义清晰的工具 schema，让 LLM 能正确调用
2. **提示工程**：设计好的系统提示，明确 Agent 的角色和规则
3. **记忆管理**：维护对话历史，实现上下文连贯
4. **错误处理**：完善的异常处理和重试机制
5. **性能优化**：合理使用缓存、批处理等技术

### 最佳实践

1. **模块化设计**：工具、Agent、Chain 分离
2. **可观测性**：完善的日志和监控
3. **安全性**：输入验证、权限控制、沙箱执行
4. **可测试性**：单元测试、集成测试
5. **可扩展性**：插件化架构、配置驱动

---

## 常见面试题

**1. 如何设计一个客服 Agent 的工具集？**

**答案要点：**
- 订单查询：查询订单状态、详情
- 物流追踪：查询配送信息
- 退款处理：检查政策、创建申请
- 人工转接：处理复杂问题
- FAQ 检索：回答常见问题

**2. RAG 系统如何提高检索准确率？**

**答案要点：**
- 查询重写：生成多个查询角度
- 混合检索：向量 + 关键词
- 重排序：使用 LLM 评估相关性
- 上下文压缩：过滤无关内容
- 反馈学习：根据用户反馈优化

**3. 如何保证 Agent 的安全性？**

**答案要点：**
- 输入验证：防止注入攻击
- 权限控制：限制工具访问范围
- 沙箱执行：代码执行隔离
- 速率限制：防止滥用
- 审计日志：记录所有操作
- 内容过滤：检测有害输出

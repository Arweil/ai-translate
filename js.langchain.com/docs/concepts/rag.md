# 检索增强生成（RAG）

[前置要求]
* [检索](https://js.langchain.com/docs/concepts/retrieval)

## 概述

检索增强生成（Retrieval Augmented Generation，RAG）是一种强大的技术，它通过将语言模型与外部知识库相结合来增强其能力。RAG解决了模型的一个关键限制：模型依赖固定的训练数据集，这可能导致信息过时或不完整。当收到查询时，RAG系统首先在知识库中搜索相关信息。然后系统将这些检索到的信息整合到模型的提示中。模型使用提供的上下文来生成对查询的响应。通过弥合大型语言模型和动态、有针对性的信息检索之间的差距，RAG成为构建更强大、更可靠的AI系统的有力技术。

## 关键概念

![RAG概念概述](https://js.langchain.com/assets/images/rag_concepts-4499b260d1053838a3e361fb54f376ec.png)

(1) **检索系统**：从知识库中检索相关信息。

(2) **添加外部知识**：将检索到的信息传递给模型。

## 检索系统

模型的内部知识通常是固定的，或者由于训练成本高昂而不经常更新。这限制了它们回答有关时事的问题或提供特定领域知识的能力。为了解决这个问题，有各种知识注入技术，如微调或持续预训练。这两种方法都成本高昂，而且往往不太适合事实检索。使用检索系统提供了几个优势：

* **最新信息**：RAG可以访问和利用最新数据，保持响应的时效性。
* **领域特定专业知识**：通过领域特定的知识库，RAG可以在特定领域提供答案。
* **减少幻觉**：基于检索到的事实来确定响应有助于最小化虚假或编造的信息。
* **成本效益高的知识集成**：RAG为昂贵的模型微调提供了一个更高效的替代方案。

[延伸阅读]

请参阅我们的[检索概念指南](https://js.langchain.com/docs/concepts/retrieval)。

## 添加外部知识

有了检索系统后，我们需要将这个系统的知识传递给模型。RAG管道通常通过以下步骤实现这一目标：

* 接收输入查询。
* 使用检索系统基于查询搜索相关信息。
* 将检索到的信息整合到发送给LLM的提示中。
* 生成利用检索上下文的响应。

例如，这里是一个简单的RAG工作流，它将信息从检索器传递给聊天模型：

```javascript
import { ChatOpenAI } from "@langchain/openai";

// 定义一个系统提示，告诉模型如何使用检索到的上下文
const systemPrompt = `你是一个问答任务的助手。
使用以下检索到的上下文来回答问题。
如果你不知道答案，就直说不知道。
最多使用三个句子，保持答案简洁。
上下文：{context}:`;

// 定义一个问题
const question =
  "LLM驱动的自主代理系统的主要组成部分是什么？";

// 检索相关文档
const docs = await retriever.invoke(question);

// 将文档组合成单个字符串
const docsText = docs.map((d) => d.pageContent).join("");

// 用检索到的上下文填充系统提示
const systemPromptFmt = systemPrompt.replace("{context}", docsText);

// 创建一个模型
const model = new ChatOpenAI({
  model: "gpt-4",
  temperature: 0,
});

// 生成响应
const questions = await model.invoke([
  {
    role: "system",
    content: systemPromptFmt,
  },
  {
    role: "user",
    content: question,
  },
]);
```

[延伸阅读]

RAG是一个深入的领域，有许多可能的优化和设计选择：

* [了解RAG管道：全面概述和历史](https://cameronwolfe.substack.com/p/understanding-rag-pipeline)
* [RAG操作指南](https://js.langchain.com/docs/guides/rag)
* [RAG教程](https://js.langchain.com/docs/tutorials/rag)
* [RAG从零开始课程：代码和视频播放列表](https://www.youtube.com/watch?v=OQm6kS9_G_k)
* [在Freecodecamp上的RAG从零开始课程](https://www.freecodecamp.org/news/building-rag-from-scratch) 
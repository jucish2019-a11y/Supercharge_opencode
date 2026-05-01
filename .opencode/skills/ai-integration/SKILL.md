---
name: ai-integration
description: Integrate LLMs and AI features — OpenAI, Anthropic, prompt engineering, RAG, embeddings, streaming, token management, and production patterns
---

## What I do

I integrate AI/LLM capabilities into applications:

- **LLM API integration** — OpenAI, Anthropic, Google Gemini, local models
- **Prompt engineering** — System prompts, few-shot examples, chain-of-thought, structured output
- **Retrieval-augmented generation (RAG)** — Embeddings, vector stores, chunking, retrieval
- **Streaming** — Server-sent events, streaming responses, real-time UI updates
- **Token management** — Counting tokens, cost estimation, context window management
- **Agent patterns** — Tool use, function calling, multi-step reasoning, autonomous agents
- **Production patterns** — Rate limiting, caching, fallbacks, observability, content moderation

## When to use me

Use this skill when:
- Adding chat or completion features to an application
- Building RAG pipelines for document Q&A
- Implementing AI agents that use tools
- Streaming LLM responses to the UI
- Managing token limits and costs
- Adding AI-powered features (summarization, classification, extraction)
- Setting up vector search and embeddings

## LLM provider selection

```
Need AI integration?
├── Best quality, easiest start?
│   └── OpenAI (GPT-4o / GPT-4o-mini)
├── Long context, analysis tasks?
│   └── Anthropic (Claude 3.5 Sonnet / Haiku)
├── Multimodal (images, video)?
│   └── Google Gemini or OpenAI GPT-4o
├── Need local/private deployment?
│   └── Ollama with open-source models (Llama 3, Mistral)
├── Cost-sensitive, high volume?
│   └── OpenAI GPT-4o-mini or Anthropic Haiku
└── Complex reasoning, coding, math?
    └── Anthropic Claude 3.5 Sonnet or OpenAI o1
```

## Basic LLM integration

### OpenAI chat completion

```ts
import OpenAI from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

async function chat(messages: Array<{ role: string; content: string }>) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      { role: 'system', content: 'You are a helpful assistant.' },
      ...messages,
    ],
    temperature: 0.7,
    max_tokens: 1024,
  });

  return response.choices[0].message.content;
}
```

### Anthropic Claude

```ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

async function chat(messages: Array<{ role: string; content: string }>) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    system: 'You are a helpful assistant.',
    messages,
  });

  return response.content[0].text;
}
```

### Provider abstraction

```ts
type Message = { role: 'system' | 'user' | 'assistant'; content: string };

interface LLMProvider {
  chat(messages: Message[], options?: LLMOptions): Promise<string>;
  chatStream(messages: Message[], options?: LLMOptions): AsyncIterable<string>;
  embed(text: string): Promise<number[]>;
}

interface LLMOptions {
  model?: string;
  temperature?: number;
  maxTokens?: number;
}

class OpenAIProvider implements LLMProvider {
  private client: OpenAI;

  constructor(apiKey: string) {
    this.client = new OpenAI({ apiKey });
  }

  async chat(messages: Message[], options?: LLMOptions): Promise<string> {
    const response = await this.client.chat.completions.create({
      model: options?.model ?? 'gpt-4o',
      messages,
      temperature: options?.temperature ?? 0.7,
      max_tokens: options?.maxTokens ?? 1024,
    });
    return response.choices[0].message.content ?? '';
  }

  async *chatStream(messages: Message[], options?: LLMOptions): AsyncIterable<string> {
    const stream = await this.client.chat.completions.create({
      model: options?.model ?? 'gpt-4o',
      messages,
      temperature: options?.temperature ?? 0.7,
      max_tokens: options?.maxTokens ?? 1024,
      stream: true,
    });

    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content;
      if (content) yield content;
    }
  }

  async embed(text: string): Promise<number[]> {
    const response = await this.client.embeddings.create({
      model: 'text-embedding-3-small',
      input: text,
    });
    return response.data[0].embedding;
  }
}
```

## Prompt engineering

### System prompt patterns

```ts
const SYSTEM_PROMPTS = {
  classifier: `You are a text classifier. Classify the user message into exactly one of these categories: {categories}.
Respond with only the category name, nothing else.`,

  extractor: `You are a data extractor. Extract the following fields from the user message: {fields}.
Respond in JSON format with the extracted fields. If a field is not found, use null.`,

  summarizer: `You are a summarizer. Summarize the user message in {length} words or less.
Focus on the key points and omit filler. Use clear, concise language.`,

  codeReviewer: `You are a code reviewer. Review the provided code for:
1. Bugs and logic errors
2. Security vulnerabilities
3. Performance issues
4. Style violations
5. Missing error handling

Format your response as a list of findings with severity levels (CRITICAL, WARNING, SUGGESTION).`,
};
```

### Few-shot examples

```ts
const messages: Message[] = [
  { role: 'system', content: 'Extract entity information from text. Respond in JSON.' },
  { role: 'user', content: 'John Smith works at Acme Corp in New York.' },
  { role: 'assistant', content: '{"name": "John Smith", "company": "Acme Corp", "location": "New York"}' },
  { role: 'user', content: 'Sarah Chen is the CTO of TechStart based in San Francisco.' },
];
```

### Structured output (JSON mode)

```ts
async function extractStructured(text: string): Promise<ExtractedData> {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    response_format: { type: 'json_object' },
    messages: [
      {
        role: 'system',
        content: `Extract data from the user message. Respond with a JSON object with these fields:
- name: string (person's name)
- company: string (company name)
- role: string (job title)
- location: string | null (city/state)
- date: string | null (any date mentioned, ISO format)`,
      },
      { role: 'user', content: text },
    ],
    temperature: 0,
  });

  return JSON.parse(response.choices[0].message.content!);
}
```

### Structured output with Zod schema

```ts
import { z } from 'zod';
import { zodTextFormat } from 'openai/helpers/zod';

const ExtractionSchema = z.object({
  name: z.string().describe("Person's name"),
  company: z.string().describe('Company name'),
  role: z.string().describe('Job title'),
  location: z.string().nullable().describe('City/location'),
});

async function extractWithSchema(text: string) {
  const response = await openai.chat.completions.parse({
    model: 'gpt-4o',
    messages: [{ role: 'user', content: text }],
    response_format: zodTextFormat(ExtractionSchema, 'extraction'),
  });

  return response.choices[0].message.parsed;
}
```

## Streaming responses

### Server-Sent Events (SSE) to client

```ts
// Server route handler
export async function POST(req: Request) {
  const { messages } = await req.json();

  const stream = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  });

  const encoder = new TextEncoder();

  const readable = new ReadableStream({
    async start(controller) {
      for await (const chunk of stream) {
        const content = chunk.choices[0]?.delta?.content;
        if (content) {
          controller.enqueue(encoder.encode(`data: ${JSON.stringify({ content })}\n\n`));
        }
      }
      controller.enqueue(encoder.encode('data: [DONE]\n\n'));
      controller.close();
    },
  });

  return new Response(readable, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      Connection: 'keep-alive',
    },
  });
}
```

### Client streaming hook

```ts
function useChatStream() {
  const [content, setContent] = useState('');
  const [isStreaming, setIsStreaming] = useState(false);

  const sendMessage = async (messages: Message[]) => {
    setContent('');
    setIsStreaming(true);

    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ messages }),
    });

    const reader = response.body!.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const text = decoder.decode(value);
      const lines = text.split('\n').filter(line => line.startsWith('data: '));

      for (const line of lines) {
        const data = line.slice(6);
        if (data === '[DONE]') break;

        try {
          const parsed = JSON.parse(data);
          setContent(prev => prev + parsed.content);
        } catch {}
      }
    }

    setIsStreaming(false);
  };

  return { content, isStreaming, sendMessage };
}
```

### Vercel AI SDK (higher-level abstraction)

```ts
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages,
    system: 'You are a helpful assistant.',
  });

  return result.toDataStreamResponse();
}

// Client hook
import { useChat } from 'ai/react';

function ChatComponent() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat();
  return (
    <div>
      {messages.map(m => <div key={m.id}>{m.content}</div>)}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} />
        <button type="submit" disabled={isLoading}>Send</button>
      </form>
    </div>
  );
}
```

## RAG (Retrieval-Augmented Generation)

### Document chunking

```ts
interface Chunk {
  id: string;
  content: string;
  metadata: {
    source: string;
    page?: number;
    section?: string;
    chunkIndex: number;
  };
}

function chunkDocument(
  text: string,
  chunkSize: number = 1000,
  overlap: number = 200,
  metadata: Partial<Chunk['metadata']> = {}
): Chunk[] {
  const chunks: Chunk[] = [];
  let start = 0;
  let index = 0;

  while (start < text.length) {
    const end = Math.min(start + chunkSize, text.length);
    const chunkText = text.slice(start, end);

    if (chunkText.trim().length > 0) {
      chunks.push({
        id: `${metadata.source}-chunk-${index}`,
        content: chunkText.trim(),
        metadata: {
          source: metadata.source ?? 'unknown',
          page: metadata.page,
          section: metadata.section,
          chunkIndex: index,
        },
      });
    }

    start += chunkSize - overlap;
    index++;
  }

  return chunks;
}
```

### Embedding and vector store

```ts
import { Pinecone } from '@pinecone-database/pinecone';

const pinecone = new Pinecone({ apiKey: process.env.PINECONE_API_KEY! });

async function indexDocuments(chunks: Chunk[]) {
  const index = pinecone.index(process.env.PINECONE_INDEX_NAME!);

  const vectors = await Promise.all(
    chunks.map(async (chunk) => {
      const embedding = await openai.embeddings.create({
        model: 'text-embedding-3-small',
        input: chunk.content,
      });

      return {
        id: chunk.id,
        values: embedding.data[0].embedding,
        metadata: chunk.metadata,
      };
    })
  );

  await index.upsert(vectors);
}

async function retrieve(query: string, topK: number = 5): Promise<Chunk[]> {
  const queryEmbedding = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: query,
  });

  const index = pinecone.index(process.env.PINECONE_INDEX_NAME!);

  const results = await index.query({
    vector: queryEmbedding.data[0].embedding,
    topK,
    includeMetadata: true,
  });

  return results.matches?.map(match => ({
    id: match.id,
    content: match.metadata?.content ?? '',
    metadata: {
      source: match.metadata?.source ?? '',
      page: match.metadata?.page,
      section: match.metadata?.section,
      chunkIndex: match.metadata?.chunkIndex ?? 0,
    },
  })) ?? [];
}
```

### RAG query pipeline

```ts
async function ragQuery(question: string): Promise<string> {
  const relevantChunks = await retrieve(question, 5);

  const context = relevantChunks
    .map(chunk => `[Source: ${chunk.metadata.source}${chunk.metadata.page ? `, p.${chunk.metadata.page}` : ''}]\n${chunk.content}`)
    .join('\n\n---\n\n');

  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `Answer the user's question based on the provided context. If the answer is not in the context, say "I don't have enough information to answer that question." Always cite the source.

Context:
${context}`,
      },
      { role: 'user', content: question },
    ],
    temperature: 0.3,
  });

  return response.choices[0].message.content ?? '';
}
```

## Function calling / Tool use

### Define tools

```ts
import OpenAI from 'openai';

const tools: OpenAI.ChatCompletionTool[] = [
  {
    type: 'function',
    function: {
      name: 'get_weather',
      description: 'Get current weather for a location',
      parameters: {
        type: 'object',
        properties: {
          location: { type: 'string', description: 'City name, e.g. "San Francisco"' },
          unit: { type: 'string', enum: ['celsius', 'fahrenheit'] },
        },
        required: ['location'],
      },
    },
  },
  {
    type: 'function',
    function: {
      name: 'search_products',
      description: 'Search the product catalog',
      parameters: {
        type: 'object',
        properties: {
          query: { type: 'string', description: 'Search query' },
          category: { type: 'string', description: 'Product category filter' },
          max_price: { type: 'number', description: 'Maximum price filter' },
        },
        required: ['query'],
      },
    },
  },
];

async function executeTool(name: string, args: Record<string, unknown>): Promise<string> {
  switch (name) {
    case 'get_weather': {
      const { location, unit } = args as { location: string; unit?: string };
      const weatherData = await fetchWeather(location, unit);
      return JSON.stringify(weatherData);
    }
    case 'search_products': {
      const { query, category, max_price } = args as { query: string; category?: string; max_price?: number };
      const results = await searchProductCatalog(query, category, max_price);
      return JSON.stringify(results);
    }
    default:
      return JSON.stringify({ error: `Unknown tool: ${name}` });
  }
}
```

### Agent loop with tool calling

```ts
async function agentChat(userMessage: string, maxIterations: number = 5): Promise<string> {
  const messages: OpenAI.ChatCompletionMessageParam[] = [
    { role: 'system', content: 'You are a helpful assistant. Use tools when needed to get accurate information.' },
    { role: 'user', content: userMessage },
  ];

  for (let i = 0; i < maxIterations; i++) {
    const response = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages,
      tools,
      tool_choice: 'auto',
    });

    const message = response.choices[0].message;
    messages.push(message);

    if (message.tool_calls) {
      for (const toolCall of message.tool_calls) {
        const args = JSON.parse(toolCall.function.arguments);
        const result = await executeTool(toolCall.function.name, args);

        messages.push({
          role: 'tool',
          tool_call_id: toolCall.id,
          content: result,
        });
      }
      continue;
    }

    return message.content ?? '';
  }

  return 'Agent reached maximum iterations without completing.';
}
```

## Token management

### Token counting

```ts
import { encoding_for_model } from 'tiktoken';

function countTokens(text: string, model: string = 'gpt-4o'): number {
  const encoding = encoding_for_model(model as any);
  const tokens = encoding.encode(text);
  encoding.free();
  return tokens.length;
}

function estimateCost(inputTokens: number, outputTokens: number, model: string = 'gpt-4o'): number {
  const pricing: Record<string, { input: number; output: number }> = {
    'gpt-4o': { input: 2.50 / 1_000_000, output: 10.00 / 1_000_000 },
    'gpt-4o-mini': { input: 0.15 / 1_000_000, output: 0.60 / 1_000_000 },
    'claude-sonnet-4-20250514': { input: 3.00 / 1_000_000, output: 15.00 / 1_000_000 },
    'claude-haiku-3-20240307': { input: 0.25 / 1_000_000, output: 1.25 / 1_000_000 },
  };

  const rates = pricing[model] ?? pricing['gpt-4o'];
  return inputTokens * rates.input + outputTokens * rates.output;
}

function trimToContextLimit(
  messages: Message[],
  systemPrompt: string,
  maxTokens: number = 128000,
  reserveForOutput: number = 4096
): Message[] {
  const systemTokens = countTokens(systemPrompt);
  const availableTokens = maxTokens - systemTokens - reserveForOutput;

  let totalTokens = 0;
  const trimmed: Message[] = [];

  for (let i = messages.length - 1; i >= 0; i--) {
    const msgTokens = countTokens(messages[i].content);
    if (totalTokens + msgTokens > availableTokens) break;
    totalTokens += msgTokens;
    trimmed.unshift(messages[i]);
  }

  return trimmed;
}
```

### Caching responses

```ts
import { createHash } from 'crypto';

interface CacheEntry {
  response: string;
  timestamp: number;
  tokens: { input: number; output: number };
}

const responseCache = new Map<string, CacheEntry>();

function cacheKey(messages: Message[], model: string, temperature: number): string {
  const content = JSON.stringify({ messages, model, temperature });
  return createHash('sha256').update(content).digest('hex');
}

async function cachedChat(
  messages: Message[],
  options: { model?: string; temperature?: number; ttlMs?: number } = {}
): Promise<string> {
  const model = options.model ?? 'gpt-4o';
  const temperature = options.temperature ?? 0.7;
  const ttlMs = options.ttlMs ?? 3600_000;

  const key = cacheKey(messages, model, temperature);
  const cached = responseCache.get(key);

  if (cached && Date.now() - cached.timestamp < ttlMs) {
    return cached.response;
  }

  const response = await openai.chat.completions.create({
    model,
    messages,
    temperature,
    max_tokens: 1024,
  });

  const result = response.choices[0].message.content ?? '';

  responseCache.set(key, {
    response: result,
    timestamp: Date.now(),
    tokens: {
      input: response.usage?.prompt_tokens ?? 0,
      output: response.usage?.completion_tokens ?? 0,
    },
  });

  return result;
}
```

## Production patterns

### Rate limiting and fallbacks

```ts
class RateLimitedLLM {
  private lastCallTime = 0;
  private minIntervalMs: number;

  constructor(
    private primary: LLMProvider,
    private fallback?: LLMProvider,
    private maxRetries: number = 3,
    requestsPerMinute: number = 60
  ) {
    this.minIntervalMs = 60_000 / requestsPerMinute;
  }

  async chat(messages: Message[], options?: LLMOptions): Promise<string> {
    for (let attempt = 0; attempt < this.maxRetries; attempt++) {
      try {
        await this.enforceRateLimit();

        const response = await this.primary.chat(messages, options);
        return response;
      } catch (error: any) {
        if (error?.status === 429) {
          const retryAfter = error?.headers?.['retry-after']
            ? parseInt(error.headers['retry-after']) * 1000
            : Math.pow(2, attempt) * 1000;

          await this.sleep(retryAfter);
          continue;
        }

        if (error?.status === 500 && this.fallback) {
          return this.fallback.chat(messages, options);
        }

        if (attempt === this.maxRetries - 1) throw error;
        await this.sleep(Math.pow(2, attempt) * 1000);
      }
    }

    throw new Error('Max retries exceeded');
  }

  private async enforceRateLimit() {
    const now = Date.now();
    const elapsed = now - this.lastCallTime;
    if (elapsed < this.minIntervalMs) {
      await this.sleep(this.minIntervalMs - elapsed);
    }
    this.lastCallTime = Date.now();
  }

  private sleep(ms: number) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### Content moderation

```ts
async function moderateContent(text: string): Promise<{ safe: boolean; flaggedcategories: string[] }> {
  const moderation = await openai.moderations.create({ input: text });

  const flagged = moderation.results[0];
  const flaggedCategories = Object.entries(flagged.categories)
    .filter(([_, flagged]) => flagged)
    .map(([category]) => category);

  return {
    safe: !flagged.flagged,
    flaggedCategories,
  };
}

async function safeChat(messages: Message[]): Promise<string> {
  const userMessages = messages.filter(m => m.role === 'user');
  const lastMessage = userMessages[userMessages.length - 1];

  const moderation = await moderateContent(lastMessage.content);
  if (!moderation.safe) {
    throw new Error(`Content flagged: ${moderation.flaggedCategories.join(', ')}`);
  }

  return chat(messages);
}
```

### Logging and observability

```ts
interface LLMLog {
  timestamp: string;
  model: string;
  inputTokens: number;
  outputTokens: number;
  latencyMs: number;
  cost: number;
  cached: boolean;
  error?: string;
}

const llmLogs: LLMLog[] = [];

async function observedChat(
  messages: Message[],
  options?: LLMOptions
): Promise<string> {
  const startTime = Date.now();
  const model = options?.model ?? 'gpt-4o';

  try {
    const response = await openai.chat.completions.create({
      model,
      messages,
      temperature: options?.temperature ?? 0.7,
      max_tokens: options?.maxTokens ?? 1024,
    });

    const log: LLMLog = {
      timestamp: new Date().toISOString(),
      model,
      inputTokens: response.usage?.prompt_tokens ?? 0,
      outputTokens: response.usage?.completion_tokens ?? 0,
      latencyMs: Date.now() - startTime,
      cost: estimateCost(
        response.usage?.prompt_tokens ?? 0,
        response.usage?.completion_tokens ?? 0,
        model
      ),
      cached: false,
    };

    llmLogs.push(log);
    return response.choices[0].message.content ?? '';
  } catch (error: any) {
    llmLogs.push({
      timestamp: new Date().toISOString(),
      model,
      inputTokens: 0,
      outputTokens: 0,
      latencyMs: Date.now() - startTime,
      cost: 0,
      cached: false,
      error: error.message,
    });
    throw error;
  }
}
```

## Quality checklist

- [ ] API keys stored in environment variables, never hardcoded
- [ ] Token limits respected — input trimmed to fit context window
- [ ] Cost estimation in place for production usage
- [ ] Streaming implemented for long responses (UX improvement)
- [ ] Rate limiting on LLM API calls (avoid 429 errors)
- [ ] Fallback model configured for resilience
- [ ] Response caching for deterministic queries (temperature: 0)
- [ ] Content moderation on user inputs
- [ ] Structured output validation (JSON schema or Zod)
- [ ] Error handling for API failures, timeouts, and empty responses
- [ ] Logging of token usage, latency, and costs per request
- [ ] PII scrubbing before sending user data to LLM APIs
- [ ] User-facing content reviewed for hallucination risk

## Anti-patterns I avoid

- Sending raw user input to LLMs without moderation or sanitization
- Ignoring token limits — requests fail or get truncated silently
- Not caching responses for repeated identical queries — wastes tokens and money
- Using temperature > 0 for factual/extraction tasks — causes inconsistency
- Trusting LLM output without validation — always parse and verify structured responses
- Hardcoding API keys in source code
- Not implementing rate limiting — LLM APIs have strict rate limits
- Using large context windows when only a few chunks are relevant — RAG first, then chat
- Ignoring streaming for conversational UIs — users see blank screens for seconds
- Not tracking token usage and costs — production bills can spike unexpectedly
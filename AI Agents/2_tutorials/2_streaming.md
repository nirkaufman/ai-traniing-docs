# Tutorial: Implementing Streaming in LangGraph Agents

## Introduction
In this tutorial, we'll learn how to implement streaming responses in LangGraph agents. Streaming is essential for creating responsive AI applications that provide real-time feedback to users.

## Prerequisites
- Basic understanding of LangGraph agents
- Knowledge of TypeScript
- Familiarity with server actions in Next.js

## Step 1: Setting Up the Basic Structure

First, let's create a server action with streaming support:

```typescript
'use server';

import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { ChatOpenAI } from "@langchain/openai";

// Basic agent setup
const llm = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0,
});

const agent = createReactAgent({
  llm,
  tools: [/* your tools here */],
});
```

## Step 2: Implementing the Streaming Function

Create a streaming function that returns a ReadableStream:

```typescript
export async function streamAgentResponse(prompt: string) {
  const agentStream = await agent.stream(
    { messages: [{ role: "user", content: prompt }] },
    { streamMode: "updates" }
  );

  return new ReadableStream({
    async start(controller) {
      for await (const chunk of agentStream) {
        // Process different types of chunks
        if (chunk.agent?.messages?.[0]?.content) {
          controller.enqueue(chunk.agent.messages[0].content);
        }
      }
      controller.close();
    }
  });
}
```

## Step 3: Handling Different Chunk Types

Let's enhance our streaming function to handle various types of chunks:

```typescript
export async function streamAgentResponse(prompt: string) {
  const agentStream = await agent.stream(
    { messages: [{ role: "user", content: prompt }] },
    { streamMode: "updates" }
  );

  return new ReadableStream({
    async start(controller) {
      try {
        for await (const chunk of agentStream) {
          // Agent messages
          if (chunk.agent?.messages?.[0]?.content) {
            controller.enqueue(chunk.agent.messages[0].content);
          }
          
          // Tool responses
          if (chunk.tools?.messages?.[0]?.content) {
            controller.enqueue(`Tool Result: ${chunk.tools.messages[0].content}`);
          }
          
          // Intermediate steps
          if (chunk.intermediate_steps) {
            for (const step of chunk.intermediate_steps) {
              if (step.action) {
                controller.enqueue(`Using tool: ${step.action.tool}`);
              }
            }
          }
        }
        controller.close();
      } catch (error) {
        controller.error(error);
      }
    }
  });
}
```

## Step 4: Using the Stream in a Client Component

Here's how to consume the stream in a client component:

```typescript
'use client';

export default function ChatComponent() {
  const handleStream = async (prompt: string) => {
    const stream = await streamAgentResponse(prompt);
    const reader = stream.getReader();

    try {
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        console.log(value); // Handle the streamed value
      }
    } catch (error) {
      console.error('Error reading stream:', error);
    }
  };

  return (
    // Your component JSX
  );
}
```

## Best Practices

1. **Error Handling**
   - Always wrap stream processing in try-catch blocks
   - Properly close streams
   - Handle errors gracefully

2. **Chunk Processing**
   - Check for chunk types before processing
   - Format messages appropriately
   - Handle multiple chunk types

3. **Performance**
   - Close streams when done
   - Process chunks efficiently
   - Clean up resources

## Next Steps
- Implement custom tool updates
- Add multiple streaming modes
- Enhance error handling
- Add progress indicators

## Resources
- [LangGraph Streaming Documentation](https://langchain-ai.github.io/langgraphjs/agents/streaming/)
- Example implementation in `actions/2_streaming.ts` 
# Building a Multi-Agent System with Supervisor Pattern

This tutorial demonstrates how to implement a supervisor pattern using LangGraph, where a supervisor agent coordinates multiple specialized agents to handle complex tasks.

## Prerequisites

- Basic understanding of TypeScript
- Familiarity with LangGraph and LangChain
- Understanding of async/await and streams

## Overview

The supervisor pattern allows you to:
- Delegate tasks to specialized agents
- Coordinate multiple agents
- Handle complex workflows
- Maintain conversation context

## Step 1: Setting Up the Project

First, import the necessary dependencies:

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { createSupervisor } from "@langchain/langgraph-supervisor";
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { tool } from "@langchain/core/tools";
import { z } from "zod";
import { MemorySaver } from "@langchain/langgraph/web";
```

## Step 2: Creating Specialized Tools

Let's create tools for hotel and flight booking:

```typescript
const bookHotel = tool(
  async (input: { hotel_name: string }) => {
    return `\n Successfully booked a stay at ${input.hotel_name}. \n`;
  },
  {
    name: "book_hotel",
    description: "Book a hotel",
    schema: z.object({
      hotel_name: z.string().describe("The name of the hotel to book"),
    }),
  }
);

const bookFlight = tool(
  async (input: { from_airport: string; to_airport: string }) => {
    return `\n Successfully booked a flight from ${input.from_airport} to ${input.to_airport}. \n`;
  },
  {
    name: "book_flight",
    description: "Book a flight",
    schema: z.object({
      from_airport: z.string().describe("The departure airport code"),
      to_airport: z.string().describe("The arrival airport code"),
    }),
  }
);
```

## Step 3: Setting Up the LLM

Configure the language model:

```typescript
const llm = new ChatOpenAI({ model: "gpt-4o" });
```

## Step 4: Creating Specialized Agents

Create agents for specific tasks:

```typescript
const flightAssistant = createReactAgent({
  llm,
  tools: [bookFlight],
  prompt: "You are a flight booking assistant",
  name: "flight_assistant",
  checkpointer: new MemorySaver()
});

const hotelAssistant = createReactAgent({
  llm,
  tools: [bookHotel],
  prompt: "You are a hotel booking assistant",
  name: "hotel_assistant",
  checkpointer: new MemorySaver()
});
```

## Step 5: Creating the Supervisor

Set up the supervisor to manage the specialized agents:

```typescript
const supervisor = createSupervisor({
  agents: [flightAssistant, hotelAssistant],
  llm,
  prompt: "You manage a hotel booking assistant and a flight booking assistant. Assign work to them, one at a time.",
}).compile();
```

## Step 6: Implementing the Stream Handler

Create a function to handle the streaming interaction:

```typescript
export async function streamSupervisorResponse(input: string) {
  const stream = await supervisor.stream({
    messages: [{
      role: "user",
      content: input,
    }, {
      role: "system",
      content: "You are a helpful assistant that can help me book a hotel and a flight. Your name is Joe."
    }],
    configurable: {
      thread_id: "1"
    }
  });

  return new ReadableStream({
    async start(controller) {
      try {
        for await (const chunk of stream) {
          // Handle supervisor messages
          if (chunk.supervisor?.messages) {
            // Get the last message from the supervisor (most recent response)
            const lastMessage = chunk.supervisor.messages[chunk.supervisor.messages.length - 1];
            if (lastMessage?.content) {
              controller.enqueue(lastMessage.content + '\n');
            }
          }

          // Handle tool messages
          if (chunk.tools?.messages) {
            for (const msg of chunk.tools.messages) {
              controller.enqueue(msg.content + '\n');
            }
          }
        }
        controller.close();
      } catch (error) {
        console.error('Streaming error:', error);
        controller.error(error instanceof Error ? error : new Error('Unknown error'));
      }
    }
  });
}
```

## Best Practices

### 1. Agent Design
- Keep agents focused on specific tasks
- Use clear, descriptive prompts
- Implement proper error handling
- Maintain conversation context

### 2. Supervisor Management
- Clear delegation of tasks
- Proper agent coordination
- Context preservation
- Error recovery

### 3. Stream Handling
- Proper message formatting
- Error handling
- State management
- Clean output formatting

### 4. State Management
- Use consistent thread IDs
- Implement proper checkpoints
- Handle conversation context
- Manage agent state

## Common Challenges

### 1. Agent Coordination
- Task delegation
- Context sharing
- State management
- Error handling

### 2. Message Flow
- Message formatting
- Stream handling
- Error recovery
- State preservation

### 3. Error Handling
- Agent errors
- Stream errors
- State errors
- Recovery mechanisms

## Next Steps

1. Add more specialized agents
2. Implement complex workflows
3. Add error recovery
4. Enhance state management

## Resources

- [LangGraph Documentation](https://langchain-ai.github.io/langgraphjs/)
- [LangChain Tools](https://js.langchain.com/docs/modules/agents/tools/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/) 
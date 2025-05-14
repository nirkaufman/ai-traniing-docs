# Building a Multi-Agent Swarm System with LangGraph

This tutorial will guide you through creating a swarm of specialized agents that can work together to handle complex tasks. We'll build a travel booking system with separate agents for flight and hotel bookings, demonstrating how to create, coordinate, and stream responses from multiple agents.

## Prerequisites

- Basic understanding of LangGraph and agents
- Familiarity with TypeScript/JavaScript
- Knowledge of streaming responses
- Understanding of tools and their implementation

## Overview

The swarm pattern allows multiple specialized agents to work together, each handling specific aspects of a task. In our example, we'll create:
- A flight booking agent
- A hotel booking agent
- A system to coordinate between them

## Project Setup

First, let's import the necessary dependencies:

```typescript
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { ChatOpenAI } from "@langchain/openai";
import { createSwarm, createHandoffTool } from "@langchain/langgraph-swarm";
import { tool } from "@langchain/core/tools";
import { z } from "zod";
import { MemorySaver } from "@langchain/langgraph/web";
import { ChatAnthropic } from "@langchain/anthropic";
```

## Creating Tools

Let's create the tools our agents will use:

```typescript
// Hotel booking tool
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

// Flight booking tool
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

## Creating Handoff Tools

We'll create tools that allow agents to transfer control to each other:

```typescript
const transferToHotelAssistant = createHandoffTool({
  agentName: "hotel_assistant",
  description: "Transfer user to the hotel-booking assistant.",
});

const transferToFlightAssistant = createHandoffTool({
  agentName: "flight_assistant",
  description: "Transfer user to the flight-booking assistant.",
});
```

## Setting Up the Language Model

We'll use Claude for our agents:

```typescript
const llm = new ChatAnthropic({
  model: "claude-3-5-sonnet-latest",
  apiKey: process.env.ANTHROPIC_API_KEY,
});
```

## Creating Specialized Agents

Now, let's create our specialized agents:

```typescript
const flightAssistant = createReactAgent({
  llm,
  tools: [bookFlight, transferToHotelAssistant],
  prompt: "You are a flight booking assistant",
  name: "flight_assistant",
  checkpointer: new MemorySaver()
});

const hotelAssistant = createReactAgent({
  llm,
  tools: [bookHotel, transferToFlightAssistant],
  prompt: "You are a hotel booking assistant",
  name: "hotel_assistant",
  checkpointer: new MemorySaver()
});
```

## Creating the Swarm

Combine the agents into a swarm:

```typescript
const swarm = createSwarm({
  agents: [flightAssistant, hotelAssistant],
  defaultActiveAgent: "flight_assistant",
}).compile();
```

## Implementing Stream Handling

Create a function to handle streaming responses:

```typescript
export async function streamSwarmResponse(input: string) {
  const stream = await swarm.stream({
    messages: [{
      role: "user",
      content: input,
    }]
  }, {
    configurable: {
      thread_id: "2"
    },
  });

  return new ReadableStream({
    async start(controller) {
      try {
        // Track processed message IDs to prevent duplicates
        const processedMessageIds = new Set<string>();

        for await (const chunk of stream) {
          // Handle flight assistant messages
          if (chunk.flight_assistant?.messages) {
            const messages = chunk.flight_assistant.messages;
            for (const msg of messages) {
              // Skip if message was already processed
              if (processedMessageIds.has(msg.id)) {
                continue;
              }
              processedMessageIds.add(msg.id);

              // Skip ToolMessage content, keep everything else
              if (msg.type !== 'tool' && msg.content) {
                // Handle content that can be string or array
                if (Array.isArray(msg.content)) {
                  // Process each content block
                  for (const block of msg.content) {
                    // Only stream text content, skip tool_use blocks
                    if (block.type === 'text' && block.text) {
                      controller.enqueue(block.text + '\n');
                    }
                  }
                } else {
                  // Handle string content
                  controller.enqueue(msg.content + '\n');
                }
              }
            }
          }
          
          // Handle hotel assistant messages
          if (chunk.hotel_assistant?.messages) {
            const messages = chunk.hotel_assistant.messages;
            for (const msg of messages) {
              // Skip if message was already processed
              if (processedMessageIds.has(msg.id)) {
                continue;
              }
              processedMessageIds.add(msg.id);

              // Skip ToolMessage content, keep everything else
              if (msg.type !== 'tool' && msg.content) {
                // Handle content that can be string or array
                if (Array.isArray(msg.content)) {
                  // Process each content block
                  for (const block of msg.content) {
                    // Only stream text content, skip tool_use blocks
                    if (block.type === 'text' && block.text) {
                      controller.enqueue(block.text + '\n');
                    }
                  }
                } else {
                  // Handle string content
                  controller.enqueue(msg.content + '\n');
                }
              }
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

## Key Features

1. **Specialized Agents**: Each agent is focused on a specific task (flight or hotel booking)
2. **Handoff Capability**: Agents can transfer control to each other when needed
3. **Message Deduplication**: Prevents duplicate messages in the stream
4. **Content Type Handling**: Supports both string and array content types
5. **Error Handling**: Robust error handling for the streaming process

## Best Practices

1. **Agent Design**:
   - Keep agents focused on specific tasks
   - Provide clear prompts and descriptions
   - Use appropriate tools for each agent

2. **Message Handling**:
   - Track message IDs to prevent duplicates
   - Handle different content types appropriately
   - Skip tool messages to maintain clean output

3. **Error Management**:
   - Implement proper error handling
   - Log errors for debugging
   - Provide meaningful error messages

## Common Challenges

1. **Message Duplication**:
   - Use message ID tracking
   - Implement proper message filtering
   - Handle message ordering correctly

2. **Content Type Handling**:
   - Support both string and array content
   - Filter out tool_use blocks
   - Maintain proper message formatting

3. **Agent Coordination**:
   - Implement clear handoff protocols
   - Maintain conversation context
   - Handle agent transitions smoothly

## Next Steps

1. Add more specialized agents
2. Implement more complex handoff scenarios
3. Add conversation history management
4. Implement retry logic for failed operations

## Resources

- [LangGraph Documentation](https://js.langchain.com/docs/modules/agents/agent_types/)
- [LangGraph Swarm Documentation](https://js.langchain.com/docs/modules/agents/agent_types/langgraph_swarm)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [ReadableStream API](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) 
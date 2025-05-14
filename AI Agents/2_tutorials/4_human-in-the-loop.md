# Human-in-the-Loop with LangGraph

This tutorial demonstrates how to implement human-in-the-loop functionality in LangGraph agents, allowing for human review and approval of sensitive operations.

## Prerequisites

- Basic understanding of TypeScript
- Familiarity with LangGraph and LangChain
- Understanding of async/await and streams

## Overview

Human-in-the-loop functionality allows you to:
- Pause agent execution for human review
- Get approval or edits for sensitive operations
- Resume execution based on human input
- Maintain conversation context across interruptions

## Step 1: Setting Up the Project

First, import the necessary dependencies:

```typescript
import { MemorySaver } from "@langchain/langgraph-checkpoint";
import { Command, interrupt } from "@langchain/langgraph";
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { tool } from "@langchain/core/tools";
import { z } from "zod";
import { ChatOpenAI } from "@langchain/openai";
```

## Step 2: Creating a Tool with Human Review

Let's create a hotel booking tool that requires human approval:

```typescript
const bookHotel = tool(
  async (input: { hotelName: string; }) => {
    let hotelName = input.hotelName;

    // Request human approval
    const response = await interrupt(
      `Trying to call \`book_hotel\` with args {'hotel_name': ${hotelName}}. ` +
      `Please approve or suggest edits.`
    );

    // Handle the response
    if (response.type === "approve") {
      return `Successfully booked a stay at ${hotelName}.`;
    } else if (response.type === "edit") {
      hotelName = response.args["hotel_name"];
      return `Successfully booked a stay at ${hotelName}.`;
    } else {
      throw new Error(`Unknown response type: ${response.type}`);
    }
  },
  {
    name: "bookHotel",
    schema: z.object({
      hotelName: z.string().describe("Hotel to book"),
    }),
    description: "Book a hotel.",
  }
);
```

Key points:
- Use `interrupt()` to pause execution and request human input
- Handle both "approve" and "edit" response types
- Return appropriate responses for each case

## Step 3: Setting Up the Agent

Configure the agent with the tool and checkpointer:

```typescript
const checkpointer = new MemorySaver();

const llm = new ChatOpenAI({
  model: "gpt-4o",
  temperature: 0,
});

const agent = createReactAgent({
  llm,
  tools: [bookHotel],
  checkpointer
});
```

## Step 4: Implementing the Stream Handler

Create a function to handle the streaming interaction:

```typescript
export async function streamWithHumanInTheLoop(input: string) {
  const config = {
    configurable: {
      thread_id: "1"
    }
  };

  const agentStream = await agent.stream(
    {
      messages: [
        {
          role: "user",
          content: input
        },
        {
          role: "system",
          content: "You are a helpful assistant that can help me book a hotel. Your name is Joe."
        }
      ]
    },
    config
  );

  return new ReadableStream({
    async start(controller) {
      try {
        for await (const chunk of agentStream) {
          // Handle interrupt messages
          if (chunk.__interrupt__) {
            controller.enqueue(chunk.__interrupt__[0].value);
            // Send default approve command
            await agent.stream(
              new Command({ resume: { type: "approve" } }),
              config
            );
            continue;
          }

          // Handle agent messages
          if (chunk.agent?.messages?.[0]?.content) {
            controller.enqueue(chunk.agent.messages[0].content);
          }

          // Handle tool messages
          if (chunk.tools?.messages) {
            for (const msg of chunk.tools.messages) {
              controller.enqueue(msg.content);
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

## Step 5: Using the Stream Handler

Here's how to use the stream handler in your client code:

```typescript
async function handleHotelBooking(input: string) {
  const stream = await streamWithHumanInTheLoop(input);
  const reader = stream.getReader();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    console.log(value);
  }
}
```

## Best Practices

1. **Error Handling**
   - Always handle both approve and edit cases
   - Include proper error messages
   - Log errors for debugging

2. **State Management**
   - Use a consistent thread_id
   - Maintain conversation context
   - Handle interruptions gracefully

3. **User Experience**
   - Provide clear approval messages
   - Handle timeouts appropriately
   - Give meaningful feedback

4. **Security**
   - Validate all inputs
   - Handle sensitive data appropriately
   - Implement proper error boundaries

## Common Challenges

1. **Tool Call Responses**
   - Ensure all tool calls have corresponding responses
   - Handle both success and error cases
   - Maintain proper message flow

2. **Stream Management**
   - Handle stream interruptions properly
   - Manage stream lifecycle
   - Clean up resources

3. **Error Recovery**
   - Implement proper error handling
   - Provide meaningful error messages
   - Allow for retry mechanisms

## Next Steps

1. Add more sophisticated approval flows
2. Implement custom response handling
3. Add more tools with human review
4. Enhance error handling and recovery

## Resources

- [LangGraph Documentation](https://langchain-ai.github.io/langgraphjs/)
- [LangChain Tools Documentation](https://js.langchain.com/docs/modules/agents/tools/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/) 
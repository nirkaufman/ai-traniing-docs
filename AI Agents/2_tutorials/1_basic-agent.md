# LangChain Basic Agent Tutorial

This tutorial explains how to create and use a basic agent with LangChain, including different variations of system prompts and memory management.

## Basic Agent Setup

The basic agent is created using the `createReactAgent` function from `@langchain/langgraph/prebuilt`. Here's how it works:

### 1. Tool Definition

First, we define a tool that our agent can use. In this case, it's a simple weather tool:

```typescript
const getWeather = tool(
    async (input: { city: string }) => {
      return `It's always sunny in ${input.city}!`;
    },
    {
      name: "getWeather",
      description: "Get weather for a given city.",
      schema: z.object({
        city: z.string().describe("The city to get the weather for"),
      }),
    }
);
```

The tool consists of:
- A function that performs the actual work
- A name and description for the tool
- A schema that defines the input parameters

### 2. LLM Configuration

Next, we set up the language model:

```typescript
const llm = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0,
})
```

### 3. Basic Agent Creation

The simplest form of the agent uses a static system prompt:

```typescript
const agent = createReactAgent({
  llm,
  tools: [getWeather],
  prompt: "You are a helpful assistant. use sarcastic tone."
});
```

### 4. Using the Basic Agent

To use the agent, we call it with a user message:

```typescript
export async function weatherAgent(prompt: string) {
  const response = await agent.invoke({
    messages: [
      {role: "user", content: prompt}
    ]
  });

  return {
    messages: response.messages.map(msg => ({
      role: msg instanceof AIMessage ? 'assistant' : 'user',
      content: msg.content
    }))
  };
}
```

## Dynamic System Prompt

For more flexibility, we can create an agent with a dynamic system prompt:

```typescript
const prompt = (
    state: typeof MessagesAnnotation.State, 
    config: RunnableConfig
): BaseMessageLike[] => {
  const userName = config.configurable?.userName;
  const systemMsg = `You are a helpful assistant. Address the user as ${userName}.`;

  return [{role: "system", content: systemMsg}, ...state.messages];
};

const agent2 = createReactAgent({
  llm,
  tools: [getWeather],
  prompt
});
```

This allows us to customize the system prompt based on runtime parameters:

```typescript
export async function weatherAgent2(userName: string, prompt: string) {
  const response = await agent2.invoke(
      {messages: [{role: "user", content: prompt}]},
      {configurable: {userName}}
  );
  // ... process response
}
```

## Adding Memory

To maintain conversation context, we can add memory to our agent:

```typescript
const checkpointer = new MemorySaver();

const agent3 = createReactAgent({
  llm,
  tools: [getWeather],
  prompt,
  checkpointer,
});
```

The agent with memory can be used like this:

```typescript
export async function weatherAgent3(userName: string, prompt: string) {
  const response = await agent3.invoke(
      {messages: [{role: "user", content: prompt}]},
      {configurable: {thread_id: 1, userName}}
  );
  // ... process response
}
```

## Key Features

1. **Tool Integration**: The agent can use custom tools to perform specific tasks
2. **Dynamic Prompts**: System prompts can be customized at runtime
3. **Memory Management**: Conversation history can be maintained using the MemorySaver
4. **Type Safety**: Full TypeScript support with proper type definitions
5. **Serialization**: Messages are properly serialized for client-server communication

## Best Practices

1. Always serialize messages before sending them to the client
2. Use proper type checking for message types
3. Implement error handling for agent responses
4. Use memory management for maintaining conversation context
5. Keep tool definitions clear and well-documented

## Next Steps

1. Add more sophisticated tools
2. Implement error handling and retry logic
3. Add conversation history management
4. Implement streaming responses
5. Add authentication and user management

# Tutorial: Implementing Tools in LangGraph Agents

## Introduction
In this tutorial, we'll learn how to implement and use tools in LangGraph agents. Tools are essential for extending agent capabilities with custom functionality.

## Prerequisites
- Basic understanding of TypeScript
- Familiarity with Zod schemas
- Knowledge of async/await

## Step 1: Basic Tool Implementation

Let's start with a simple weather tool:

```typescript
import { tool } from "@langchain/core/tools";
import { z } from "zod";

const weatherTool = tool(
  async (input: { city: string }) => {
    return `The weather in ${input.city} is sunny and 72°F!`;
  },
  {
    name: "getWeather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);
```

## Step 2: Tool with State and Updates

Let's create a more advanced tool that uses state and provides updates:

```typescript
import { LangGraphRunnableConfig } from "@langchain/langgraph";

const weatherToolWithUpdates = tool(
  async (input: { city: string }, config: LangGraphRunnableConfig) => {
    // Provide progress updates
    config.writer?.(`Fetching weather for ${input.city}...`);
    
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    return `The weather in ${input.city} is sunny and 72°F!`;
  },
  {
    name: "getWeather",
    description: "Get weather for a given city.",
    schema: z.object({
      city: z.string().describe("The city to get the weather for"),
    }),
  }
);
```

## Step 3: Tool with Direct Return

Sometimes you want to return the tool result directly:

```typescript
const calculatorTool = tool(
  async (input: { a: number; b: number }) => {
    return input.a + input.b;
  },
  {
    name: "calculator",
    description: "Add two numbers.",
    schema: z.object({
      a: z.number().describe("First number"),
      b: z.number().describe("Second number"),
    }),
    returnDirect: true
  }
);
```

## Step 4: Tool with Error Handling

Let's add proper error handling:

```typescript
const safeCalculatorTool = tool(
  async (input: { expression: string }) => {
    try {
      // Note: In a real application, use a proper expression parser
      const result = eval(input.expression);
      return `The result is ${result}`;
    } catch (error) {
      return `Error calculating expression: ${error.message}`;
    }
  },
  {
    name: "calculator",
    description: "Calculate a mathematical expression.",
    schema: z.object({
      expression: z.string().describe("The expression to calculate"),
    }),
  }
);
```

## Step 5: Integrating Tools with Agent

Now let's create an agent that uses these tools:

```typescript
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { ChatOpenAI } from "@langchain/openai";

const llm = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0,
});

const agent = createReactAgent({
  llm,
  tools: [weatherTool, calculatorTool, safeCalculatorTool],
  prompt: "You are a helpful assistant that can check the weather and do math."
});
```

## Best Practices

1. **Tool Design**
   - Keep tools focused and single-purpose
   - Use clear, descriptive names
   - Provide detailed descriptions
   - Include example usage in descriptions

2. **Schema Definition**
   - Use descriptive field names
   - Add helpful descriptions
   - Use appropriate Zod types
   - Include validation rules

3. **Error Handling**
   - Always handle potential errors
   - Provide meaningful error messages
   - Use try-catch blocks
   - Consider fallback options

4. **Performance**
   - Keep tools lightweight
   - Use async/await properly
   - Consider caching for expensive operations
   - Provide progress updates for long operations

## Next Steps
- Implement more complex tools
- Add tool combinations
- Implement tool-specific error handling
- Add tool-specific logging

## Resources
- [LangGraph Tools Documentation](https://langchain-ai.github.io/langgraphjs/agents/tools/)
- Example implementation in `actions/3_tools.ts` 
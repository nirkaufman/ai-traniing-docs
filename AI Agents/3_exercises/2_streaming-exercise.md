# Exercise: Implementing Streaming in LangGraph Agents

## Objective
Create a streaming agent that can handle multiple types of responses and provide real-time feedback to users.

## Requirements

### 1. Basic Setup
Create a server action that implements streaming for a LangGraph agent with the following tools:
- A weather tool that provides weather information
- A calculator tool that performs basic math operations
- A joke tool that tells jokes

### 2. Streaming Implementation
Implement a streaming function that:
- Returns a ReadableStream
- Handles different types of chunks (agent messages, tool responses, intermediate steps)
- Properly formats and streams each type of response
- Includes error handling

### 3. Client Integration
Create a client component that:
- Consumes the streaming response
- Displays different types of messages differently
- Shows a loading state while streaming
- Handles errors gracefully

## Tasks

### Task 1: Tool Implementation
```typescript
// Implement these tools
const weatherTool = tool(/* ... */);
const calculatorTool = tool(/* ... */);
const jokeTool = tool(/* ... */);
```

### Task 2: Streaming Function
```typescript
export async function streamAgentResponse(prompt: string) {
  // Implement streaming logic
}
```

### Task 3: Client Component
```typescript
export default function StreamingChat() {
  // Implement client-side streaming consumption
}
```

## Evaluation Criteria

1. **Functionality (40%)**
   - All tools work correctly
   - Streaming works for all message types
   - Error handling is implemented

2. **Code Quality (30%)**
   - Clean code structure
   - Proper TypeScript types
   - Efficient stream handling

3. **User Experience (30%)**
   - Smooth streaming experience
   - Clear message formatting
   - Proper loading states
   - Error feedback

## Bonus Challenges

1. **Multiple Streaming Modes**
   - Implement both "updates" and "messages" streaming modes
   - Allow switching between modes

2. **Custom Tool Updates**
   - Add progress indicators for long-running tools
   - Implement custom update messages

3. **Enhanced Error Handling**
   - Add retry logic for failed streams
   - Implement graceful degradation

## Submission
Submit your solution with:
1. Server action implementation
2. Client component implementation
3. Any additional components or utilities
4. A brief explanation of your implementation choices

## Resources
- [LangGraph Streaming Documentation](https://langchain-ai.github.io/langgraphjs/agents/streaming/)
- Example implementation in `actions/2_streaming.ts`
- Reference implementation in `actions/3_tools.ts` 
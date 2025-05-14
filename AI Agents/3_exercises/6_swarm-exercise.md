# Exercise: Building a Multi-Agent Travel Assistant

## Overview
In this exercise, you will build a travel assistant system using the swarm architecture pattern. The system will consist of multiple specialized agents working together to handle different aspects of travel planning.

## Learning Objectives
- Implement a swarm of specialized agents
- Create and manage agent handoffs
- Handle streaming responses from multiple agents
- Implement proper message handling and deduplication

## Requirements

### 1. Basic Setup
Create a new file `actions/6_swarm.ts` with the following imports:
```typescript
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { createSwarm, createHandoffTool } from "@langchain/langgraph-swarm";
import { tool } from "@langchain/core/tools";
import { z } from "zod";
import { MemorySaver } from "@langchain/langgraph/web";
import { ChatAnthropic } from "@langchain/anthropic";
```

### 2. Tool Implementation
Implement the following tools:
1. `bookFlight`: Books a flight with departure and arrival airports
2. `bookHotel`: Books a hotel with hotel name
3. `transferToHotelAssistant`: Handoff tool to hotel assistant
4. `transferToFlightAssistant`: Handoff tool to flight assistant

### 3. Agent Creation
Create two specialized agents:
1. Flight Assistant:
   - Uses `bookFlight` and `transferToHotelAssistant` tools
   - Focused on flight booking tasks
   - Clear prompt for flight booking

2. Hotel Assistant:
   - Uses `bookHotel` and `transferToFlightAssistant` tools
   - Focused on hotel booking tasks
   - Clear prompt for hotel booking

### 4. Swarm Implementation
Create a swarm that:
- Combines both agents
- Sets flight assistant as default
- Properly compiles the swarm

### 5. Stream Handling
Implement a `streamSwarmResponse` function that:
- Takes user input as a string
- Creates a stream from the swarm
- Handles messages from both agents
- Prevents message duplication
- Handles different content types
- Implements proper error handling

## Tasks

### Task 1: Tool Creation
```typescript
// TODO: Implement bookFlight tool
const bookFlight = tool(
  // Your implementation here
);

// TODO: Implement bookHotel tool
const bookHotel = tool(
  // Your implementation here
);

// TODO: Implement handoff tools
const transferToHotelAssistant = createHandoffTool({
  // Your implementation here
});

const transferToFlightAssistant = createHandoffTool({
  // Your implementation here
});
```

### Task 2: Agent Configuration
```typescript
// TODO: Configure language model
const llm = new ChatAnthropic({
  // Your implementation here
});

// TODO: Create flight assistant
const flightAssistant = createReactAgent({
  // Your implementation here
});

// TODO: Create hotel assistant
const hotelAssistant = createReactAgent({
  // Your implementation here
});
```

### Task 3: Swarm Creation
```typescript
// TODO: Create and compile swarm
const swarm = createSwarm({
  // Your implementation here
}).compile();
```

### Task 4: Stream Handling
```typescript
export async function streamSwarmResponse(input: string) {
  // TODO: Implement stream handling
  // Your implementation here
}
```

## Success Criteria
1. Tools are properly implemented with correct schemas
2. Agents are configured with appropriate tools and prompts
3. Swarm is properly created and compiled
4. Stream handling:
   - Correctly processes messages from both agents
   - Prevents message duplication
   - Handles different content types
   - Implements proper error handling

## Bonus Challenges
1. Add a third agent for attraction booking
2. Implement conversation history management
3. Add retry logic for failed operations
4. Implement more complex handoff scenarios

## Testing
Test your implementation with the following scenarios:
1. Simple flight booking request
2. Simple hotel booking request
3. Combined flight and hotel booking request
4. Error handling scenarios

## Submission
Submit your implementation in `actions/6_swarm.ts`. Make sure to:
1. Include all required imports
2. Implement all required tools
3. Configure agents properly
4. Implement stream handling
5. Include proper error handling
6. Add comments explaining your implementation

## Resources
- [LangGraph Documentation](https://js.langchain.com/docs/modules/agents/agent_types/)
- [LangGraph Swarm Documentation](https://js.langchain.com/docs/modules/agents/agent_types/langgraph_swarm)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/) 
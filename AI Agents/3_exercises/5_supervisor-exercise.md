# Exercise: Building a Multi-Agent Travel Assistant

## Objective
Create a travel assistant system using the supervisor pattern, where a supervisor agent coordinates multiple specialized agents to handle different aspects of travel planning.

## Requirements

### 1. Basic Implementation
- Create a supervisor agent that coordinates multiple specialized agents
- Implement at least two specialized agents (e.g., flight booking and hotel booking)
- Set up proper message handling and state management
- Implement streaming responses

### 2. Specialized Agents
Each agent should:
- Have a specific focus area
- Include appropriate tools
- Handle its own error cases
- Maintain conversation context

### 3. Tools Implementation
Create tools for:
- Flight booking
- Hotel booking
- Additional travel-related services (at least one more)

### 4. Advanced Features
- Implement proper error handling
- Add progress updates
- Include input validation
- Handle state management

## Tasks

### 1. Basic Setup
- Set up the project structure
- Import necessary dependencies
- Configure the language model
- Create the supervisor agent

### 2. Specialized Agents
Create at least two specialized agents:
- Flight booking agent
- Hotel booking agent
- Additional agent (your choice)

### 3. Tool Implementation
Implement the following tools:
- Flight booking tool
- Hotel booking tool
- Additional tool (your choice)

### 4. Supervisor Integration
- Create the supervisor agent
- Configure agent coordination
- Implement message handling
- Set up state management

### 5. Stream Handling
- Implement streaming responses
- Handle different message types
- Manage error cases
- Format output properly

## Evaluation Criteria

### 1. Code Quality (30%)
- Clean code structure
- Proper TypeScript usage
- Error handling
- Documentation

### 2. Functionality (40%)
- Supervisor implementation
- Agent coordination
- Tool functionality
- Stream handling

### 3. Advanced Features (20%)
- Error handling
- State management
- Progress updates
- Input validation

### 4. Documentation (10%)
- Code comments
- README
- API documentation
- Usage examples

## Bonus Challenges

### 1. Advanced Features
- Add more specialized agents
- Implement complex workflows
- Add authentication
- Include rate limiting

### 2. Performance
- Optimize response time
- Implement caching
- Add retry mechanisms
- Handle concurrent requests

### 3. User Experience
- Add progress indicators
- Implement better error messages
- Include validation feedback
- Add response formatting

## Submission Requirements

### 1. Code Implementation
- Complete TypeScript implementation
- All required tools
- Supervisor and agent setup
- Stream handling

### 2. Documentation
- Code comments
- README file
- API documentation
- Usage examples

### 3. Testing
- Basic functionality tests
- Error handling tests
- Stream handling tests
- Integration tests

## Resources

### 1. Documentation
- [LangGraph Supervisor Documentation](https://langchain-ai.github.io/langgraphjs/)
- [LangChain Tools Documentation](https://js.langchain.com/docs/modules/agents/tools/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)

### 2. Example Code
- Basic supervisor setup
- Tool implementations
- Stream handling
- Error handling

### 3. Implementation Tips
- Start with basic setup
- Add features incrementally
- Test thoroughly
- Document as you go 
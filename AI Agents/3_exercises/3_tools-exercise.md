# Exercise: Building a Travel Assistant with Tools

## Objective
Create a travel assistant agent that helps users plan their trips by implementing various tools for weather, flights, hotels, and attractions.

## Requirements

### 1. Basic Tools Implementation
Implement the following tools with proper schemas and error handling:

1. **Weather Tool**
   - Input: destination and month
   - Output: Weather forecast
   - Schema: Validate destination and month format
   - Error handling: Invalid location

2. **Flight Tool**
   - Input: origin, destination, and date
   - Output: Available flights
   - Schema: Validate airport codes and date format
   - Error handling: Invalid airports or dates

3. **Hotel Tool**
   - Input: arrival date
   - Output: Available hotels
   - Schema: Validate date format
   - Error handling: Invalid dates

4. **Attraction Tool**
   - Input: destination
   - Output: Popular attractions
   - Schema: Validate destination
   - Error handling: Invalid location

### 2. Advanced Features
Enhance the basic tools with:

1. **Progress Updates**
   - Add progress messages for long operations
   - Show estimated completion time
   - Provide status updates

2. **Input Validation**
   - Validate all input parameters
   - Provide helpful error messages
   - Handle edge cases

3. **Error Handling**
   - Implement try-catch blocks
   - Provide meaningful error messages
   - Add fallback options

4. **Response Formatting**
   - Format responses consistently
   - Include relevant details
   - Make output user-friendly

## Tasks

### Task 1: Basic Tool Implementation
1. Create the weather tool with:
   - Proper TypeScript types
   - Zod schema
   - Error handling
   - Example usage

2. Create the flight tool with:
   - Input validation
   - Response formatting
   - Error handling
   - Example usage

3. Create the hotel tool with:
   - Date validation
   - Response formatting
   - Error handling
   - Example usage

4. Create the attraction tool with:
   - Location validation
   - Response formatting
   - Error handling
   - Example usage

### Task 2: Tool Enhancement
1. Add progress updates to all tools
2. Implement comprehensive error handling
3. Add input validation
4. Improve response formatting

### Task 3: Agent Integration
1. Create an agent with all tools
2. Implement proper tool selection
3. Handle tool responses
4. Format final output

## Evaluation Criteria

### 1. Code Quality (40%)
- Proper TypeScript types
- Clean code structure
- Error handling
- Documentation

### 2. Tool Implementation (30%)
- Schema definition
- Input validation
- Response formatting
- Error handling

### 3. Agent Integration (20%)
- Tool selection
- Response handling
- Error recovery
- Output formatting

### 4. Documentation (10%)
- Code comments
- Tool descriptions
- Example usage
- Error messages

## Bonus Challenges

### 1. Advanced Features
- Add caching for frequently accessed data
- Implement rate limiting
- Add authentication
- Create tool combinations

### 2. Performance
- Optimize API calls
- Implement request queuing
- Add response caching
- Improve error recovery

### 3. User Experience
- Add rich responses
- Implement suggestions
- Add confirmation steps
- Provide fallback options

## Submission Requirements

1. **Code**
   - All tool implementations
   - Agent configuration
   - Error handling
   - Example usage

2. **Documentation**
   - Tool descriptions
   - Schema definitions
   - Error handling
   - Example responses

3. **Testing**
   - Input validation
   - Error handling
   - Response formatting
   - Edge cases

## Resources
- [LangGraph Tools Documentation](https://langchain-ai.github.io/langgraphjs/agents/tools/)
- [Zod Documentation](https://zod.dev/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- Example implementation in `actions/3_tools.ts`

## Tips
1. Start with basic tool implementation
2. Add error handling early
3. Test with various inputs
4. Document as you go
5. Use TypeScript for type safety
6. Follow best practices
7. Test edge cases
8. Format responses consistently 
# Exercise: Building a Human-in-the-Loop Travel Booking System

## Objective
Create a travel booking system that requires human approval for sensitive operations like hotel bookings, flight reservations, and payment processing.

## Requirements

### Basic Requirements
1. **Hotel Booking Tool**
   - Input: hotel name, dates, room type
   - Requires human approval
   - Handles both approval and edit responses
   - Returns booking confirmation

2. **Flight Booking Tool**
   - Input: origin, destination, dates
   - Requires human approval
   - Handles both approval and edit responses
   - Returns flight confirmation

3. **Payment Processing Tool**
   - Input: amount, currency, payment method
   - Requires human approval
   - Handles both approval and edit responses
   - Returns payment confirmation

### Advanced Features
1. **State Management**
   - Maintain conversation context
   - Handle multiple bookings
   - Track approval status

2. **Error Handling**
   - Handle invalid inputs
   - Manage approval timeouts
   - Recover from errors

3. **User Experience**
   - Clear approval messages
   - Meaningful feedback
   - Smooth flow

## Tasks

### 1. Basic Implementation
1. Create the hotel booking tool with human approval
   ```typescript
   const bookHotel = tool(
     async (input: { hotelName: string; dates: string; roomType: string; }) => {
       // Implement hotel booking with human approval
     }
   );
   ```

2. Create the flight booking tool with human approval
   ```typescript
   const bookFlight = tool(
     async (input: { origin: string; destination: string; dates: string; }) => {
       // Implement flight booking with human approval
     }
   );
   ```

3. Create the payment processing tool with human approval
   ```typescript
   const processPayment = tool(
     async (input: { amount: number; currency: string; paymentMethod: string; }) => {
       // Implement payment processing with human approval
     }
   );
   ```

### 2. Stream Handler Implementation
1. Create a stream handler for the agent
   ```typescript
   export async function streamBookingAgentResponse(input: string) {
     // Implement stream handler with human approval
   }
   ```

2. Handle different message types
   - Interrupt messages
   - Agent messages
   - Tool messages

3. Implement error handling
   - Invalid inputs
   - Approval timeouts
   - Stream errors

### 3. State Management
1. Implement thread management
   ```typescript
   const config = {
     configurable: {
       thread_id: string
     }
   };
   ```

2. Handle conversation context
   - Maintain booking state
   - Track approvals
   - Manage edits

### 4. Error Recovery
1. Implement error boundaries
   - Input validation
   - Approval handling
   - Stream management

2. Add recovery mechanisms
   - Retry logic
   - Fallback options
   - Error messages

## Evaluation Criteria

### Code Quality (40%)
- Proper TypeScript usage
- Clean code structure
- Error handling
- Documentation

### Functionality (30%)
- Tool implementation
- Stream handling
- State management
- Error recovery

### User Experience (20%)
- Clear messages
- Smooth flow
- Error feedback
- Approval process

### Documentation (10%)
- Code comments
- README
- Usage examples
- Error handling guide

## Bonus Challenges

### 1. Advanced Features
- Implement custom response types
- Add approval timeouts
- Create approval UI
- Add booking history

### 2. Performance
- Optimize stream handling
- Improve state management
- Reduce latency
- Handle concurrent requests

### 3. Security
- Add input validation
- Implement rate limiting
- Add authentication
- Secure sensitive data

## Submission Requirements

### Code
- Complete implementation
- TypeScript types
- Error handling
- Documentation

### Documentation
- README with setup instructions
- Usage examples
- Error handling guide
- Testing instructions

### Testing
- Unit tests
- Integration tests
- Error scenarios
- Edge cases

## Resources

### Documentation
- [LangGraph HITL Guide](https://langchain-ai.github.io/langgraphjs/agents/human-in-the-loop/)
- [LangChain Tools](https://js.langchain.com/docs/modules/agents/tools/)
- [TypeScript Async/Await](https://www.typescriptlang.org/docs/handbook/async-await.html)

### Example Code
```typescript
// Basic tool structure
const tool = tool(
  async (input: InputType) => {
    const response = await interrupt(message);
    if (response.type === "approve") {
      return successMessage;
    } else if (response.type === "edit") {
      return editedSuccessMessage;
    }
  }
);

// Stream handler structure
export async function streamHandler(input: string) {
  const stream = await agent.stream(input);
  return new ReadableStream({
    async start(controller) {
      // Handle stream
    }
  });
}
```

## Tips
1. Start with basic tool implementation
2. Add human approval step by step
3. Test each component separately
4. Handle errors early
5. Document as you go 
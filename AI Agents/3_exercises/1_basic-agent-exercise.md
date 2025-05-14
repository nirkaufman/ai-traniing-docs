# Basic Agent Exercise

## Overview
In this exercise, you'll create a simple agent that can help users find information about movies. You'll implement different versions of the agent, from basic to more advanced features.

## Prerequisites
- Basic understanding of TypeScript
- Familiarity with LangChain concepts
- Node.js and npm installed

## Exercise 1: Basic Movie Agent
Create a basic agent that can provide information about movies.

### Tasks:
1. Create a new file `movie-agent.ts` in your project
2. Implement a basic tool for getting movie information:
   ```typescript
   const getMovieInfo = tool(
     async (input: { title: string }) => {
       // Return mock movie information
       return `Movie: ${input.title}\nGenre: Action\nRating: 4.5/5`;
     },
     {
       name: "getMovieInfo",
       description: "Get information about a movie",
       schema: z.object({
         title: z.string().describe("The title of the movie"),
       }),
     }
   );
   ```
3. Set up the LLM and create a basic agent
4. Implement the agent function to handle user queries

### Expected Output:
When a user asks "Tell me about The Matrix", the agent should respond with movie information.

## Exercise 2: Dynamic Prompt Agent
Enhance the movie agent to use dynamic prompts based on user preferences.

### Tasks:
1. Create a new function for dynamic prompt generation
2. Modify the agent to use the dynamic prompt
3. Add user preferences to the configuration
4. Test with different user preferences

### Example:
```typescript
const prompt = (
  state: typeof MessagesAnnotation.State,
  config: RunnableConfig
): BaseMessageLike[] => {
  const userPreference = config.configurable?.preference;
  const systemMsg = `You are a movie expert. Focus on ${userPreference} aspects of movies.`;
  return [{role: "system", content: systemMsg}, ...state.messages];
};
```

## Exercise 3: Memory-Enabled Agent
Add conversation memory to the movie agent.

### Tasks:
1. Implement the MemorySaver
2. Add conversation history tracking
3. Test the agent's ability to reference previous conversations

### Example Usage:
```typescript
const checkpointer = new MemorySaver();
const agent = createReactAgent({
  llm,
  tools: [getMovieInfo],
  prompt,
  checkpointer,
});
```

## Exercise 4: Advanced Features
Add more sophisticated features to your movie agent.

### Tasks:
1. Add multiple tools:
   - Movie recommendations
   - Actor information
   - Similar movies
2. Implement error handling
3. Add response formatting

## Testing Your Implementation

### Test Cases:
1. Basic Query:
   ```typescript
   const response = await movieAgent("Tell me about The Matrix");
   ```

2. Dynamic Prompt:
   ```typescript
   const response = await movieAgent2("user123", "Tell me about The Matrix", "action");
   ```

3. Memory Test:
   ```typescript
   const response = await movieAgent3("user123", "What other movies are similar?", "action");
   ```

## Evaluation Criteria

Your implementation will be evaluated based on:
1. Code organization and structure
2. Proper type usage
3. Error handling
4. Response quality
5. Memory management
6. Tool implementation

## Bonus Challenges

1. **Streaming Responses**: Implement streaming for real-time responses
2. **Multiple Tools**: Add more sophisticated movie-related tools
3. **Context Management**: Implement better context handling for complex queries
4. **Error Recovery**: Add retry logic for failed requests
5. **Response Formatting**: Implement structured response formatting

## Submission

Create a new branch and submit your implementation with:
1. Source code
2. Test cases
3. Documentation
4. Example usage

## Resources

- [LangChain Documentation](https://js.langchain.com/docs/)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Next.js Documentation](https://nextjs.org/docs)

## Tips

1. Start with the basic implementation and gradually add features
2. Use TypeScript's type system to catch errors early
3. Test each feature as you implement it
4. Keep your code modular and well-documented
5. Use proper error handling throughout your implementation 
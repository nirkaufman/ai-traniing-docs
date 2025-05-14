# Tutorial: Building a Travel Booking Agent with Tools

## Introduction
In this tutorial, we'll learn how to build a travel booking agent using LangGraph tools. We'll create various tools for weather forecasting, flight booking, hotel reservations, and more.

## Prerequisites
- Basic understanding of TypeScript
- Familiarity with LangGraph and LangChain
- Knowledge of async/await and server actions

## Step 1: Setting Up the Project

First, let's import the necessary dependencies:

```typescript
import { z } from "zod";
import { ChatOpenAI } from "@langchain/openai";
import { tool } from "@langchain/core/tools";
import { createReactAgent } from "@langchain/langgraph/prebuilt";
import { MemorySaver } from "@langchain/langgraph/web";
```

## Step 2: Implementing Basic Tools

Let's create our first tool for weather forecasting:

```typescript
type WeatherForecastArgs = {
  destination: string,
  month: string
}

const weatherForecast = tool(
  async ({destination, month}: WeatherForecastArgs) => {
    return `Weather forecast for ${destination} in ${month}: Sunny and warm!`
  },
  {
    name: "weather-forecast",
    description: "provides a weather forecast for a destination in a month",
    schema: z.object({
      destination: z.string().describe("destination to get weather forecast for, e.g. London, UK"),
      month: z.string().describe("month to get weather forecast for, e.g. January, February, March, etc.")
    })
  }
)
```

## Step 3: Adding More Tools

Let's add a flight locator tool:

```typescript
type FlightLocatorArgs = {
  destination: string
  date: string,
  origin: string,
}

const flightLocator = tool(
  async ({destination, date, origin}: FlightLocatorArgs) => {
    return `Flight locator from ${origin} to ${destination} in ${date}: ELAL #657, departure at 10:00 AM, arrival at 11:00 AM`;
  },
  {
    name: "flight-locator",
    description: "provides a flight locator for a given origin and destination in a month",
    schema: z.object({
      origin: z.string().describe("origin city to get flight locator for, e.g. London, UK"),
      destination: z.string().describe("destination city to get flight locator for, e.g. New York, USA"),
      date: z.string().describe("date to get flight locator for, e.g. January 12th, February 1st, March 4, etc.")
    })
  }
)
```

## Step 4: Creating Additional Tools

Let's add tools for airport taxi booking, hotel booking, and attraction recommendations:

```typescript
const airportTaxiBooking = tool(
  async ({origin, departure}: {origin: string, departure: string}) => {
    return `Taxi booking to ${origin} airport at ${departure}`;
  },
  {
    name: "airport-taxi-booking",
    description: "provides a taxi booking for a given origin, to get on time for your flight departure",
    schema: z.object({
      origin: z.string().describe("origin city to get taxi booking for, e.g. Tel-Aviv, Israel"),
      departure: z.string().describe("departure time to get taxi booking for, e.g. 10:00 AM, 11:00 AM, 12:00 PM, etc."),
    })
  }
)

const hotelBooking = tool(
  async ({arrivalDate}: {arrivalDate: string}) => {
    return `Hotel booking for ${arrivalDate}`;
  },
  {
    name: "hotel-booking",
    description: "provides a hotel booking for a given arrival date",
    schema: z.object({
      arrivalDate: z.string().describe("arrival date to get hotel booking for, e.g. January 12th, February 1st, March 4, etc."),
    })
  }
)

const attractionRecommendation = tool(
  async ({destination}: {destination: string}) => {
    return `Attraction recommendation in ${destination}`;
  },
  {
    name: "attraction-recommendation",
    description: "provides an attraction recommendation",
    schema: z.object({
      destination: z.string().describe("destination to get attraction recommendations for")
    })
  }
)
```

## Step 5: Setting Up the Agent

Now, let's create our agent with all the tools:

```typescript
const llm = new ChatOpenAI({
  model: "gpt-4o",
  temperature: 0,
});

const checkpointer = new MemorySaver();

const agent = createReactAgent({
  llm,
  checkpointer,
  tools: [
    weatherForecast,
    flightLocator,
    airportTaxiBooking,
    hotelBooking,
    attractionRecommendation
  ],
});
```

## Step 6: Implementing Streaming Response

Let's create a server action that streams the agent's responses:

```typescript
interface ToolMessage {
  content: string;
  name: string;
  tool_call_id: string;
}

export async function streamBookingAgentResponse(input: string) {
  const agentStream = await agent.stream(
    { 
      messages: [
        { role: "user", content: input },
        {
          role: "system",
          content: "You are a helpful assistant that can help me book a flight, hotel, and attraction. your name is Joe"
        }
      ] 
    },
    {
      configurable: {
        thread_id: 1,       
      }
    }
  );

  return new ReadableStream({
    async start(controller) {
      try {
        for await (const chunk of agentStream) {
          // Handle agent messages
          if (chunk.agent?.messages?.[0]?.content) {
            controller.enqueue(chunk.agent.messages[0].content);
          }
          
          // Handle tool messages
          if (chunk.tools?.messages) {
            chunk.tools.messages.forEach((msg: ToolMessage) => {
              controller.enqueue(msg.content);
            });
          }
        }
        controller.close();
      } catch (error) {
        controller.error(error);
      }
    }
  });
}
```

## Best Practices

1. **Tool Design**
   - Keep tools focused on specific tasks
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

4. **Streaming Implementation**
   - Handle both agent and tool messages
   - Proper error handling
   - Clean stream closure
   - Type safety for messages

## Next Steps

1. Add more sophisticated tools
2. Implement error handling and retry logic
3. Add conversation history management
4. Implement authentication
5. Add rate limiting and caching

## Resources
- [LangGraph Tools Documentation](https://langchain-ai.github.io/langgraphjs/agents/tools/)
- [LangChain Streaming Documentation](https://js.langchain.com/docs/modules/agents/agent_types/react)
- Example implementation in `actions/3_tools.ts` 
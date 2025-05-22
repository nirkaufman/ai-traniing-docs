# Exercise Simple LLM Interaction


## Setup
- install the OpenAI integration: `npm i @langchain/openai`
- Create a directory named `server`
- create a `.env` file at the root folder
- Add your `OpenAI` API key as an environment variable: `OPENAI_API_KEY=your-api-key`
- **Dont forget to add it to `.gitignore`** !  


## Instantiate the model
- under `server` create a file named: `chat-action.ts`
- Implement simple model integration:

__chat-action.ts__
```typescript
'use server'

import { ChatOpenAI } from "@langchain/openai";
import { HumanMessage, SystemMessage } from "@langchain/core/messages";

const model = new ChatOpenAI({ model: "gpt-4" });

const messages = [
  new SystemMessage("Translate the following from English into spanish"),
  new HumanMessage("hi!"),
];


export async function chatResponse() {
  const response = await model.invoke(messages);
  return JSON.stringify(response, null, 2);
}
```

## Execute from the client

- in `chat/page.tsx` refactor the `handleSubmit` function:

__page.tsx__
```tsx
import {chatResponse} from "@/server/chat-action";

const handleSubmit = async (e: FormEvent) => {
  e.preventDefault()

  const response = await chatResponse();
  console.log(response);
}
```

- type something in the input to trigger the handler
- inspect the console and explore the response structure


## Stream response back to client

- since chat modes are `runnable`, we can stream the response back to the client
- create a new function in `chat-action.ts` and add the following: 

__server/chat.ts__
```typescript
export async function streamChat(prompt: string) {
  
  const messages = [
    new SystemMessage("Translate the following from English into spanish"),
    new HumanMessage(prompt),
  ];
  
  const stream = await model.stream(messages);

  // We need to use a ReadableStream for server-sent events
  return new ReadableStream({
    async start(controller) {
      for await (const chunk of stream) {
        // Send each chunk to the client
        controller.enqueue(chunk.content);
      }
      controller.close();
    },
  });
}
```

- Now let's refactor `chat/page.tsx` to use the streamed response.
- Start by adding a utility function to create a unique id at the tp of the file


___chat/page.tsx__
```tsx
const generateId = (() => {
  let counter = 0;
  return () => `${Date.now()}-${counter++}`;
})();
```

- then, replace the `useState` statements with these:

___chat/page.tsx__
```tsx
  const [messages, setMessages] = useState<Message[]>([]);
  const [inputValue, setInputValue] = useState('');
  const [isStreaming, setIsStreaming] = useState(false);
```


- finally, re-implement the `handleSubmit` function


__chat/page.tsx__
```tsx
const handleSubmit = async (e: FormEvent) => {
    e.preventDefault()

    if (!inputValue.trim() || isStreaming) return

    // Add a user message to the chat with unique ID
    const userMessage: Message = {
      id: generateId(),
      role: 'user',
      content: inputValue.trim()
    }

    setMessages(prev => [...prev, userMessage])

    // Store user input and clear the form
    const userPrompt = inputValue

    setInputValue('')
    setIsStreaming(true)

    try {
      // Add an empty AI message that will be populated with streaming content
      const aiMessageId = generateId()

      setMessages(prev => [...prev, {
        id: aiMessageId,
        role: 'ai',
        content: ''
      }])

      // Start streaming
      const stream = await streamChat(userPrompt)
      const reader = stream.getReader()

      // Process the stream
      while (true) {
        const { done, value } = await reader.read()

        if (done) break

        // Update the AI message content with each chunk
        setMessages(prev => {
          const aiMessage = prev.find(msg => msg.id === aiMessageId)
          if (!aiMessage) return prev

          return prev.map(msg =>
              msg.id === aiMessageId
                  ? { ...msg, content: msg.content + value }
                  : msg
          )
        })
      }
    } catch (error) {
      console.error('Streaming error:', error)

      // Add an error message with unique ID
      setMessages(prev => [...prev, {
        id: generateId(),
        role: 'ai',
        content: 'Sorry, there was an error processing your request.'
      }])
    } finally {
      setIsStreaming(false)
    }
  }
```

- the `isStreaming` boolean can be used on the `input` an `button` in `JSX` 

__chat/page.tsx__
```tsx
      'use client'

import { useState, useRef, useEffect, FormEvent } from 'react'
import {streamChat, streamChatWithPromptTemplate} from "@/server/chat-action";

type Message = {
  id: string
  role: 'user' | 'ai'
  content: string
}

// Helper function to generate unique IDs
const generateId = (() => {
  let counter = 0;
  return () => `${Date.now()}-${counter++}`;
})();

export default function ChatPage() {
  const [messages, setMessages] = useState<Message[]>([])
  const [inputValue, setInputValue] = useState('')
  const [isStreaming, setIsStreaming] = useState(false)

  const messagesEndRef = useRef<HTMLDivElement>(null)

  // Automatically scroll to the bottom when messages change
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' })
  }, [messages])

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault()

    if (!inputValue.trim() || isStreaming) return

    // Add a user message to the chat with unique ID
    const userMessage: Message = {
      id: generateId(),
      role: 'user',
      content: inputValue.trim()
    }

    setMessages(prev => [...prev, userMessage])

    // Store user input and clear the form
    const userPrompt = inputValue

    setInputValue('')
    setIsStreaming(true)

    try {
      // Add an empty AI message that will be populated with streaming content
      const aiMessageId = generateId()

      setMessages(prev => [...prev, {
        id: aiMessageId,
        role: 'ai',
        content: ''
      }])

      // Start streaming
      const stream = await streamChat(userPrompt)
      const reader = stream.getReader()

      // Process the stream
      while (true) {
        const { done, value } = await reader.read()

        if (done) break

        // Update the AI message content with each chunk
        setMessages(prev => {
          const aiMessage = prev.find(msg => msg.id === aiMessageId)
          if (!aiMessage) return prev

          return prev.map(msg =>
              msg.id === aiMessageId
                  ? { ...msg, content: msg.content + value }
                  : msg
          )
        })
      }
    } catch (error) {
      console.error('Streaming error:', error)

      // Add an error message with unique ID
      setMessages(prev => [...prev, {
        id: generateId(),
        role: 'ai',
        content: 'Sorry, there was an error processing your request.'
      }])
    } finally {
      setIsStreaming(false)
    }
  }

  const handleWithTemplate = async (e: FormEvent) => {
    e.preventDefault()

    if (!inputValue.trim() || isStreaming) return

    const userMessage: Message = {
      id: generateId(),
      role: 'user',
      content: inputValue.trim()
    }

    setMessages(prev => [...prev, userMessage])
    const userPrompt = inputValue
    const browserLanguage = window.navigator.language.split('-')[0]

    setInputValue('')
    setIsStreaming(true)

    try {
      const aiMessageId = generateId()
      setMessages(prev => [...prev, {
        id: aiMessageId,
        role: 'ai',
        content: ''
      }])

      const stream = await streamChatWithPromptTemplate(browserLanguage, userPrompt)
      const reader = stream.getReader()

      while (true) {
        const {done, value} = await reader.read()
        if (done) break

        setMessages(prev => {
          const aiMessage = prev.find(msg => msg.id === aiMessageId)
          if (!aiMessage) return prev

          return prev.map(msg =>
              msg.id === aiMessageId
                  ? {...msg, content: msg.content + value}
                  : msg
          )
        })
      }
    } catch (error) {
      console.error('Streaming error:', error)
      setMessages(prev => [...prev, {
        id: generateId(),
        role: 'ai',
        content: 'Sorry, there was an error processing your request.'
      }])
    } finally {
      setIsStreaming(false)
    }
  }

  return (
      <div className="flex flex-col h-screen">
        {/* Messages container */}
        <div className="flex-1 overflow-y-auto px-4 py-5">
          <div className="max-w-2xl mx-auto">
            {messages.length === 0 ? (
                <div className="text-center text-gray-500 py-10">
                  Start a conversation by typing a message below
                </div>
            ) : (
                messages.map((message) => (
                    <div
                        key={message.id}
                        className={`mb-4 p-4 rounded-lg ${
                            message.role === 'user'
                                ? 'border-white border mr-auto max-w-[80%]'
                                : 'border-orange-400 border ml-auto max-w-[80%]'
                        }`}
                    >
                      <div className="text-sm font-semibold mb-1">
                        {message.role === 'user' ? 'You:' : 'AI:'}
                      </div>
                      <div className="whitespace-pre-wrap">{message.content}</div>
                    </div>
                ))
            )}
            <div ref={messagesEndRef} />
          </div>
        </div>

        {/* Input form fixed at the bottom */}
        <div className="border-t border-gray-200 bg-black">
          <div className="max-w-2xl mx-auto p-4">
            <form onSubmit={handleSubmit} className="flex gap-2">
              <input
                  autoFocus
                  type="text"
                  value={inputValue}
                  onChange={(e) => setInputValue(e.target.value)}
                  placeholder="Type your message here..."
                  className="flex-1 p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2"
                  disabled={isStreaming}
              />
              <button
                  type="submit"
                  className={`bg-black text-white border border-orange-400 px-4 py-2 rounded-lg ${!isStreaming ? 'hover:bg-orange-400' : 'opacity-50'} focus:outline-none cursor-pointer`}
                  disabled={isStreaming}
              >
                {isStreaming ? 'Sending...' : 'Send'}
              </button>
            </form>
          </div>
        </div>
      </div>
  )
}

```







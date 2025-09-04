---
title: "From Text to Charts: Generative UI using assistant-ui"
seoTitle: "How to Build Generative UI with assistant-ui"
seoDescription: "Build Generative UI with Assistant UI to turn natural language queries into charts using MCP, makeAssistantTool, and makeAssistantToolUI."
datePublished: Mon Jul 07 2025 14:35:20 GMT+0000 (Coordinated Universal Time)
cuid: cmct7duhr001f02la04ich94z
slug: from-text-to-charts-generative-ui-using-assistant-ui
tags: data-visualization, generative-ui, llm-tools, assistant-ui

---

## What is Generative UI?

**Generative UI** refers to user interfaces that are *not fixed ahead of time* but are instead generated dynamically — often by an AI model — in response to user queries, application state, or real-time data.

## Why Generative UI at all?

I was working on an LLM project to enable users to query a database using natural language  
(*more on this in a separate post on MCP: Model Context Protocol*).

At first, displaying results as **Markdown tables** worked well.

But I realized if the assistant was going to be used by **non-technical users**, it needed to go beyond rows and columns. Tables show data, but **charts make insights instantly clear** — especially for questions like:

* “Which loan type has the most applications?”
    
* “What’s the breakdown of applications by status?”
    

This is where **Generative UI** comes into play — that is dynamically generating UI components based on intent, not hardcoded logic.

## Putting Generative UI into Practice

At its core, **Generative UI** is about shifting from *static* interfaces to *adaptive, AI-driven experiences*. Rather than designing every possible UI screen ahead of time, we let the system **generate the interface dynamically** based on user intent, data context, and goals.

As described in [this excellent overview on Medium](https://medium.com/design-bootcamp/generative-ui-the-future-of-dynamic-user-experiences-880b1781fcf4), building a generative UI requires three fundamental capabilities:

### 1\. Understanding Intent

The system must deeply understand what the user wants — not just in keywords, but semantically. For example:  
“Show me the number of applications by type” should be interpreted as a request for a **grouped count**, not just a text summary.

### 2\. Selecting the Right Interaction Pattern

Once the intent is known, the UI needs to choose the **most effective way to present it**:

* Table
    
* Chart
    
* Form
    
* Summary
    

This requires mapping intent to a suitable component.

### 3\. Generating and Rendering UI Elements Dynamically

Finally, the system must be able to **construct and display** the chosen interface, in context — ideally within a fluid conversational or task-driven flow.

### How assistant-ui Fulfills These Requirements

**assistant-ui** makes it practical to implement all three layers of Generative UI, without building everything from scratch:

* ✅ **Intent understanding** is handled via language models — enhanced by tool descriptions and structured schemas.
    
* ✅ **Interaction mapping** is enabled through **tool definitions** using `makeAssistantTool`, where each tool maps to a specific capability (e.g., draw a chart).
    
* ✅ **UI generation** happens with `makeAssistantToolUI`, which binds the tool’s output to a fully interactive visual component — rendered directly in the chat interface.
    

Execution is powered by `streamText`, which allows the assistant to:

1. Parse the user’s question
    
2. Call the appropriate backend tool (e.g., via MCP)
    
3. Invoke a rendering tool (e.g., `drawBarchart`)
    
4. Stream the result back into the assistant window as a chart or component
    

### Example: Let the Assistant Render Visual Charts

Here’s what we’ll build:

* A `drawBarchart` tool to visualize `{ key, value }` grouped data
    
* A corresponding chart renderer using `makeAssistantToolUI`
    
* The same for `drawPiechart`
    
* Assistant instructions that guide the AI to use these tools automatically
    

### Step 1: Define Chart UI with `makeAssistantToolUI`

The UI layer connects tool results to React visual components. Here’s how I built it:

**Bar Chart Tool UI**

```typescript
export const GetBarchartToolUI = makeAssistantToolUI<{}, 
    { key: string; value: number }[]>({
  toolName: "drawBarchart",
  render: ({ result, status }) => {
    if (status.type === "running") {
      return <div>Loading summary data...</div>;
    }
    if (!Array.isArray(result) || result.length === 0) {
      return <div>No data found.</div>;
    }

    return (
      <div className="w-full h-72">
        <ResponsiveContainer>
          <BarChart data={result}>
            <CartesianGrid strokeDasharray="3 3" />
            <XAxis dataKey="key" />
            <YAxis />
            <Tooltip />
            <Bar dataKey="value" fill="#4f46e5" />
          </BarChart>
        </ResponsiveContainer>
      </div>
    );
  },
});
```

### Step 2: Define the Tool Logic

Using `makeAssistantTool`, I exposed a tool the assistant can call when it needs to draw a chart.

```typescript
const drawBarchart = tool({
  description: "Draw a bar chart from tabular data",
  parameters: z.object({
    data: z.array(
      z.object({
        key: z.string(),
        value: z.number(),
      })
    ),
  }),
  execute: async ({ data }) => {
    return data;
  },
});
```

You can create a similar `drawPiechart` tool the same way.

---

### Step 3: Register Tools in the Chat Runtime

This is where it all comes together: wiring the tool into the AI assistant runtime.

```typescript
export async function POST(req: Request) {
  const session = await auth();
  const jwt = await session.getToken();

  const mcpClient = await createMCPClient({...});

  const mcpTools = await mcpClient.tools();
  const { messages, system, tools } = await req.json();

  const result = streamText({
    model: azure("gpt-4o"),
    messages,
    toolCallStreaming: true,
    system,
    tools: {
      ...frontendTools(tools),
      ...mcpTools,
      drawBarchart,
    },
    onError: console.log,
  });

  return result.toDataStreamResponse();
}
```

---

## 🧠 Step 4: Instruct the Assistant on Chart Use

assistant-ui gives you the ability to guide the model’s behavior via `useAssistantInstructions`.

```typescript
useAssistantInstructions(`
  You are an intelligent assistant that helps users query a financial services database.
  You ONLY answer questions that can be answered using the provided database schema.

  If the user asks to visualize data (e.g., applications by type), first use the MCP tool 
  to group the results. Then call the "drawBarchart" tool and pass in the grouped data 
  using the { key, value } format.

  Avoid general financial advice or any data not available in the MCP schema.
`);
```

### Example in Action

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1751898609786/3f1b5b35-efac-4ba7-aa62-9389d05bc5e8.png align="center")

**User:**  
"Can you show me the number of loan applications by status?"

**Assistant does:**

1. Queries database using MCP to get the grouped result.
    
2. Receives:
    
    ```typescript
    [{ "key": "Approved", "value": 140 }, { "key": "Declined", "value": 75 }]
    ```
    
3. Calls `drawBarchart`.
    
4. The UI renders a live, interactive bar chart in the chat.
    

That’s Generative UI at work — dynamically transforming natural language into charts, based entirely on real-time context and data.

**References**

* [Assistant UI – Official Documentation](https://www.assistant-ui.com/docs)  
    Core docs on `makeAssistantTool`, `makeAssistantToolUI`, and building copilots.
    
* [makeAssistantTool Guide](https://www.assistant-ui.com/docs/copilots/make-assistant-tool)  
    Defines how to expose structured tools for the assistant to call.
    
* [makeAssistantToolUI Guide](https://www.assistant-ui.com/docs/copilots/make-assistant-tool-ui)  
    Shows how to render tool results as dynamic UI elements.
    
* [Tool UI Integration Guide](https://www.assistant-ui.com/docs/guides/ToolUI)  
    High-level overview of how Assistant UI enables generative interfaces.
    
* [Generative UI: The Future of Dynamic User Experiences](https://medium.com/design-bootcamp/generative-ui-the-future-of-dynamic-user-experiences-880b1781fcf4)  
    A great conceptual introduction to generative UI patterns and their design implications.
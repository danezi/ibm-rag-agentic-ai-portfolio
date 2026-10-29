# 08 · Agentic AI with LangGraph, CrewAI, AutoGen and BeeAI

[Course on Coursera](https://www.coursera.org/learn/agentic-ai-with-langgraph-crewai-autogen-and-beeai) · Status: 🟡 In progress (~55%)

## What I built

**With LangGraph:** the core LLM workflow patterns. Basic patterns: a job application assistant that chains a resume summary into a cover letter, a router that sends each request to the right handler, and a translator that produces three languages in parallel. Advanced patterns: an orchestrator-worker meal planner that starts one "chef" worker per dish, and an evaluator-optimizer loop that revises an investment plan until its risk grade matches the investor's profile.

**With CrewAI:** teams of role-based agents. A research crew that searches the web and writes a blog post plus social media posts; a five-agent meal and grocery planner that returns validated, structured data; and a restaurant support chatbot built two ways to compare tools on the agent with tools on the task.

## Architecture

**Meal & grocery planner (CrewAI)**

```mermaid
flowchart LR
    IN["Meal, servings,<br/>budget, restrictions"] --> A1["Meal planner<br/>(web search)"]
    A1 -->|MealPlan| A2["Shopping organizer"]
    A2 -->|GroceryShoppingPlan| A3["Budget advisor"]
    A3 --> A4["Leftover manager<br/>(YAML-configured)"]
    A4 --> A5["Report compiler"]
    A5 --> OUT(["Meal planning report"])
```

**Evaluator-optimizer (investment plan)**

```mermaid
flowchart LR
    P["Investor profile"] --> T["Determine<br/>target risk grade"]
    T --> G["Generator<br/>(writes / revises plan)"]
    G --> EV["Evaluator<br/>(grades risk,<br/>gives feedback)"]
    EV --> R{"Grade = target<br/>or limit reached?"}
    R -->|no, feedback| G
    R -->|yes| OUT(["Final plan"])
```

**Orchestrator-worker (meal guide)**

```mermaid
flowchart LR
    M["Meal list"] --> O["Orchestrator<br/>(structured dish list)"]
    O -->|"Send: one task<br/>per dish"| W1["Chef worker 1"]
    O --> W2["Chef worker 2"]
    O --> W3["Chef worker n"]
    W1 --> S["Synthesizer<br/>(merged guide)"]
    W2 --> S
    W3 --> S
```

**Request router**

```mermaid
flowchart LR
    Q["User request"] --> R["Router<br/>(tool calling,<br/>Pydantic schema)"]
    R -->|ride_hailing_call| A["Ride request<br/>summary"]
    R -->|restaurant_order| B["Order summary"]
    R -->|groceries| C["Grocery list"]
    R -->|no match| D["Default handler"]
```

## Projects & labs

| Project | What it does | Key techniques | Stack |
|---|---|---|---|
| Workflow patterns | Job application assistant, summarize/translate router, parallel translator | Prompt chaining, routing with conditional edges, parallel branches with an aggregator | LangGraph, LangChain, GPT-4o mini |
| Request router | Sends requests to ride-hailing, restaurant, grocery or fallback handlers | Tool-calling classifier with a Pydantic model, fallback route | LangGraph, LangChain, GPT-4o mini |
| **Meal guide (orchestrator-worker)** | Splits a meal list into dishes and writes a cooking guide per dish | Structured output, dynamic fan-out with the `Send` API, reducer to collect results, synthesizer | LangGraph, LangChain, GPT-4o mini |
| **Investment plan optimizer** | Writes an investment plan and revises it until its risk grade matches the investor profile | Evaluator-optimizer loop, structured grading, feedback-driven regeneration, iteration limit | LangGraph, LangChain, GPT-4o mini |
| **Research & content crew** | A researcher agent searches the web, a writer agent turns the findings into a blog post, a social media agent writes short posts | CrewAI agents (role, goal, backstory), sequential tasks, web search tool, token usage tracking | CrewAI, Serper, watsonx.ai (Llama 4 Maverick) |
| **Meal & grocery planner** | Five agents plan a meal, write a grocery list by store section, check the budget, suggest leftover uses and compile a report | Pydantic output models per task, YAML-configured agents (`@CrewBase`), web search | CrewAI, Serper, watsonx.ai (Granite 4) |
| **Restaurant support chatbot** | Answers customer questions from the restaurant's FAQ PDF, built agent-centric and task-centric for comparison; plus a calculator agent with custom tools | Tools on agents vs. tools on tasks, PDF search, custom `@tool` functions | CrewAI, watsonx.ai (Granite 4) |

## Results

- **Investment plan optimizer:** target grade "aggressive" for a 29-year-old high-risk investor. The grade swung from "high risk" to "moderate" and back before matching "aggressive" in the 6th evaluation, the last one the iteration limit allowed.
- **Meal guide:** the planner and the chef workers were tested step by step (for example, a Spaghetti Bolognese walkthrough from an "Italian chef" worker).
- **Request router:** all four test requests reached the right handler, including "What's the weather like today?" → fallback handler.
- **Parallel translator:** one sentence returned in French, Spanish and Japanese in a single run.
- **Research & content crew:** produced a blog post on generative AI adoption with sourced figures. The two-agent run used about 106,000 tokens, roughly 90 % of them input (instructions, search results, previous task output).
- **Meal & grocery planner:** for "Chicken Stir Fry, 4 people, $25, no nuts, low sodium" the crew returned a recipe, a grocery list grouped by store section with prices, saving tips and leftover ideas. One leftover idea suggested peanut sauce despite the "no nuts" restriction.
- **Restaurant support chatbot:** both versions answered from the FAQ PDF (phone number, opening hours); the calculator agent returned 15 for "add 4, 5, and 6" and 5040 for "multiply 7 and 8 also 9 dont forget 10".

## Key learnings

- Basic patterns cover most multi-step workflows: chaining for dependent steps, routing for different request types, parallelization for independent subtasks.
- The `Send` API makes fan-out dynamic: the number of workers depends on the data, not on the graph definition.
- In an evaluator-optimizer loop, feedback can overcorrect and make the result oscillate around the target; an explicit iteration limit is what guarantees the loop ends.
- A router needs a fallback route, and a compiled LangGraph app is a snapshot: changes only take effect after compiling again.
- In CrewAI, sequential tasks hand their output to the next agent as context; that is convenient but makes token costs grow with every step.
- Pydantic output models turn agent results into validated data; constraints that live only in a prompt (like an allergy) can still be ignored by a later agent and need an explicit check.
- Tools on the agent give flexibility (the agent chooses); tools on the task give control (each step uses a fixed source). For a support bot that must answer from official documents, the task-centric design is safer.

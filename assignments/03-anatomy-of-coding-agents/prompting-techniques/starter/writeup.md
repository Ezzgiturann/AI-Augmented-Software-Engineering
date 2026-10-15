# Week 3 — Prompting Techniques: Writeup

All four checks pass locally with `llama3.1:8b` via Ollama (`temperature: 0.0`).

| Technique | File | Result |
|---|---|---|
| K-shot prompting | `k_shot_prompting.py` | 4/4 |
| Chain-of-thought | `chain_of_thought.py` | 3/3 |
| Tool calling | `tool_calling.py` | PASS |
| RAG | `rag.py` | 3/3 |

---

## 1. K-shot prompting — ticket classification

### Final prompt
```text
You are a support ticket classifier for an online store.

Classify the customer's message into exactly ONE of these labels:
billing, shipping, returns, technical, other

Rules:
- Reply with ONLY the label.
- Lowercase, no punctuation, no explanation.

Examples:
Message: I was charged the wrong amount on my invoice.
Label: billing

Message: My order still hasn't arrived.
Label: shipping

Message: I would like to send this item back for a refund.
Label: returns

Message: The mobile app keeps crashing whenever I try to log in.
Label: technical
```

### Sample input / output
- **Input:** `My card was charged twice for order 4412.`
- **Output:** `billing`

### What I learned
Examples and rules do different jobs: examples teach which label fits a message, while
rules lock the output format, so a correct label with extra words or punctuation would
still fail.

---

## 2. Chain-of-thought — refund eligibility

### Final prompt
```text
You are a refund eligibility checker for an online store.

Think step by step. For each rule in the policy below:
1. Check whether the purchase was made within the last 30 days.
2. Check whether the product is unused.
3. Check whether the product is marked as final-sale.

Then approve the refund only if all three rules are satisfied. If even one rule is violated, reject the refund. If information is missing, do not assume that the rule is satisfied. If any rule is violated, the decision must be NO, even if other information is missing.

Explain your reasoning briefly by checking each rule individually.

The LAST line of your answer must be exactly:
DECISION: YES
or
DECISION: NO
```
(The refund policy is appended to this prompt by the script.)

### Sample input / output
- **Input:** `Bought 45 days ago, unused.`
- **Output:**
  ```text
  1. The purchase was made 45 days ago, which is outside the 30-day window. (Rule not satisfied)
  2. The product is unused, which satisfies this rule.
  3. No information is provided about the product being marked as final-sale, but since we don't have enough information to assume it's not final-sale, we can't satisfy this rule.

  DECISION: NO
  ```

### What I learned
Checking each rule separately made the decision easy to follow and verify, but treating
missing information as a rejection is too strict: an eligible customer who never mentions
final-sale would be refused.

---

## 3. Tool calling — order lookup

### Final prompt
```text
You are a customer support assistant for an online store.

When a customer asks about the status, location, or delivery progress of an order, you must always call the order status lookup tool before answering.

Never guess, invent, or assume an order's status. Use the order ID provided by the customer, extracting only the numeric part and removing any prefixes such as '#' or 'order'.

After receiving the tool's result, answer the customer using the retrieved information. If the customer has not provided an order ID, ask them to provide it before calling the tool.
```

### Tool schema descriptions
- **Tool:** "Retrieves the current status of a customer's order. Use this tool whenever the customer asks about an order's status, location, delivery progress, or estimated arrival time."
- **`order_id`:** "The customer's order ID. Extract only the numeric part from the customer's message, removing prefixes such as '#' or 'order'. For example, if the customer says 'order 4412' or '#4412', pass '4412'."

### Sample input / output
- **Input:** `Hi, can you check on order 4412 for me?`
- **Output (tool call):** `get_order_status(order_id="4412")`
- **Tool result:** `shipped, arriving Friday`

### What I learned
The model doesn't fetch data itself; it only decides which function to call and with what
arguments. The tool description tells it *when* to call, and the parameter description
helps it extract a clean order ID.

---

## 4. RAG — policy-grounded answers

### Retriever
Keyword overlap: lowercase and tokenize with `re.findall(r"[a-z]+", ...)`, drop stop
words and words shorter than 3 characters, then keep every chunk that shares at least
one word with the question (prefix match, so `ship` ↔ `shipping`, `return` ↔ `returns`).

### Final prompt
```text
You are a helpful customer support assistant.

Answer the user's question using only the information provided in the Context.

Do not use outside knowledge or make up information that is not present in the Context.

If the Context does not contain enough information to answer the question, say "I don't know."

Keep your answer clear, concise, and relevant to the question.
```

### Sample input / output
- **Input:** `Do you ship internationally?`
- **Retrieved chunks:** 3 (both Shipping lines + the Billing line, via `ships`)
- **Output:** `yes, we offer international shipping to most countries.`

- **Input:** `What's your CEO's home address?`
- **Retrieved chunks:** 0
- **Output:** `i don't know.`

### What I learned
RAG has two separate steps, retrieval and generation. My first retriever returned five
irrelevant chunks for the CEO question; the prompt still prevented a made-up answer, but a
good prompt can only reduce the damage of bad retrieval, not fix it.

---

## Closing: why LLMs are good at structured tasks — and where they need scaffolding

A prompt is more than telling the model what to do: how the instructions and examples are
written changes the results. LLMs follow patterns and formats well, so a label, a
`DECISION:` line, or a tool call is easy for them once it is clearly specified. Prompts
alone are not enough when the model needs real data or the right document, which is
where tools and retrieval come in. Each technique solves a different problem, and testing
the prompts was what exposed their weaknesses and showed me how to improve them.

---
layout: post
title:  "Conversation Design: The Prompt Graph"
date:   2024-03-19 01:00:00 -0500
categories: jekyll update
regenerate: true
---

Today, creating chatbots that handle complex tasks naturally yet remain under our control is a significant challenge. Traditional graph-based chatbots offer control but can feel rigid and unnatural, while modern LLM (Large Language Model) chatbots excel in naturalness but can veer off course in complex interactions. This article introduces a hybrid approach we’re calling a “prompt graph”, combining the best of both worlds: the precision of graph-based systems with the fluid conversation style of LLMs. Our goal is to provide a solution that makes chatbots both more engaging for users and easier to manage for developers, especially in intricate tasks. We'll explore how this method can transform chatbot interactions, making them feel more natural without sacrificing control.

## Old school: Graph based Chatbots

In the pre-LLM days conversations were built using intent detection and directed graphs. The way it starts is a conversation designer builds a graph structure where all of the nodes are the things you want the chatbot to say. The graph edges that connect the nodes determine how the conversation is allowed to proceed. During the conversation a NLU (natural language understanding) classifier is used to detect the user intent and that intent is used to choose which node is next. Throughout the conversation, the algorithm that’s managing the dialogue is also able to detect entities in the conversation and store them in memory slots to be used later.

### Graph-Based Dialogue System Example: Ordering a Pizza or Salad

|![Pizza Bot](/img/prompt-graph/graph_1.png)|
|:--:| 
| *Pizza Ordering Graph Diagram* |

**Nodes (Chatbot Responses):**
- Greeting: "Hello! Welcome to PizzaSaladBot. Are you looking to order a pizza or a salad today?"
- Choose Pizza Type: "What type of pizza would you like? We have Pepperoni, Margherita, and Veggie."
- Choose Salad Type: "What type of salad would you prefer? We offer Caesar, Greek, and Garden."
- Choose Size (Pizza): "What size pizza would you like? We offer Small, Medium, and Large."
- Add Dressing (Salad): "Would you like to add any dressings? We have Italian, Ranch, and Vinaigrette."
- Confirmation: "Would you like to add anything else to your order?"
- Checkout: "Great! Your order is on its way. Would you like to proceed to checkout?"
- Farewell: "Thank you for ordering with PizzaSaladBot. Your order will arrive in 30 minutes. Goodbye!"

**Intents/Edges (User Intentions):**
- Intent to Order Pizza: Leads from the "Greeting" node to the "Choose Pizza Type" node.
- Intent to Order Salad: Leads from the "Greeting" node to the "Choose Salad Type" node.
- Choose Pizza Type: Moves the conversation from the "Choose Pizza Type" node to the "Choose Size (Pizza)" node. One ML model is trained to understand what size of pizza they want: small, medium, large. Another model is trained to determine what kind of pizza it is: Pepperoni, Margherita, or Veggie
- Choose Salad Type: Transitions from the "Choose Salad Type" node to the "Add Dressing (Salad)" node. Similar to above, a ML model is trained to detect what dressing they want.
- Add More or Checkout: This decision point either loops back to adding more items or moves forward to the "Checkout" node.
Confirm Order: Moves the conversation from the "Checkout" node to the "Farewell" node.

### Example Conversation Flow:

---

**User**: "Hi"
```
[Slots: ]
```

**Bot**: "Hello! Welcome to PizzaSaladBot. Are you looking to order a pizza or a salad today?"

**User**: "I want a salad."

```
[Intent: Salad] [Slots: salad]
```

**Bot**: "What type of salad would you prefer? We offer Caesar, Greek, and Garden."

**User**: "Greek, please."

```
[Intent: greek] [Slots: salad, salad:greek]
```

**Bot**: "Would you like to add any dressings? We have Italian, Ranch, and Vinaigrette."

**User**: "Add Italian dressing."

```
[Intent: italian] [Slots: salad, salad:greek, dressing:italian]
```

**Bot**: "Would you like to add anything else to your order?"

**User**: "No, thanks."

```
[Intent: no] [Slots: salad, salad:greek, dressing:italian]
```

**Bot**: "Great! Your order is on its way. Would you like to proceed to checkout?"

**User**: "Yes."

```
[Intent: yes] [Slots: salad, salad:greek, dressing:italian]
```

**Bot**: "Thank you for ordering with PizzaSaladBot. Your order will arrive in 30 minutes. Goodbye!"

---
<div>
<br>
<br>
</div>

## New school: Single prompt LLM-based Chatbots

Now that LLMs are easy to use, they provide some really nice functionality for chatbots.

Here's a prompt you might use with GPT for the same conversation as above:

```md
You are an AI designed to assist customers in ordering food from a virtual restaurant that offers both pizzas and salads. When interacting with customers, follow these guidelines:

1. Greet the customer and ask if they would like to order a pizza or a salad today.

2. If the customer wants to order a pizza:

  Ask them to choose the type of pizza (Pepperoni, Margherita, Veggie).

  After they choose, ask for the size (Small, Medium, Large).

  Confirm their pizza order and ask if they would like to add anything else.

3. If the customer wants to order a salad:

  Ask them to choose the type of salad (Caesar, Greek, Garden).

  After choosing, ask if they would like to add any dressings (Italian, Ranch, Vinaigrette).

  Confirm their salad order and ask if they would like to add anything else.

4. Handle additional orders by looping back to the initial options if they want to add more items.

5. Once the customer is done, proceed to checkout. Confirm the order and provide an estimated delivery time.

6. Thank the customer and end the conversation.

Remember to maintain a polite and helpful tone throughout the conversation. Your goal is to ensure the customer's experience is pleasant and efficient.
```

The concept of intents and nodes is eliminated. For these chatbots, the conversation is generated dynamically and the prompt provides guidance for the chatbot.

### Issues with these Approaches

For small, well-scoped conversations like ordering pizza, graph-based chatbots can be a great solution. For larger, more subtle and sophisticated use cases, they feel quite mechanical and fake.

Conversely, when conversations get long, or prompts get sufficiently complex, LLM-based chatbots tend to get lost and do not handle errors very well. It is very hard to build reliable guardrails into LLM-based chatbots.
The Prompt Graph: A Hybrid Approach


## The Prompt Graph: A Hybrid Approach

Ideally, we are able to combine the best from both styles. We want to use the natural language and flexibility from LLM-based chatbots, but also provide guidance or enforce some guardrails on the conversation to have better control over it. To do this we're going to take ideas from both worlds. Essentially, we want to create a graph-like structure for the dialogue, but allow an LLM to control the transition between nodes as well as generate the chatbot dialogue on the fly.

So, the way we go about it is to create a graph of prompts. Each prompt node includes instructions on what the goals of the node are, the context of where things are in the conversation, and how to decide what to do next. Wrapping those nodes is code that applies any restrictions on the behavior we want.

### The Graph Structure

|![Pizza Bot](/img/prompt-graph/graph_2.png)|
|:--:| 
| *Pizza Ordering Prompt Graph Diagram* |

For this conversation we can simplify the graph based conversation into fewer nodes:

    'Greeting'
    'Order Pizza'
    'Order Salad'
    'Checkout'

Because we are using an LLM to handle conversational aspects of each node, we don't need to create each individual utterance as a node in the graph. For instance, the 'Order Pizza' node can handle choosing a side, toppings, and all the other options.

In this approach we essentially use the LLM as a pretrained intent detection and dialogue generator. We can guide the output of the LLM using the prompt but are freed from explicitly handing all of the edge cases. We have the option to include instructions on how to handle conversation repairs (Digressions, Corrections, Cancellations, Chitchat, Completion, Clarification, Cannot Handle, Human Handoff) but are not required to. If we include broad guidance like Be polite, helpful, and concise then most of the time, the chatbot will do something "reasonable" to handle trips off the happy path.

### Prompt Node Components

In the prompt graph model, each node comprises three key components:

- Prompt: A dynamic string template, populated with conversational data, which serves as the input for the LLM.
- JSON Output: A predefined schema that outlines the structured format expected from the LLM's response. This schema simplifies parsing by clearly defining the data structure, aiding in the efficient extraction of elements such as memory fields, next node navigation, chatbot responses, and any additional requirements.
- Additional Constraints: Custom business logic and rules applied within the node to ensure output validity and adherence to multi-step logic, which LLMs may not inherently manage.

The use of JSON output schemas facilitates more straightforward and error-resistant parsing, as opposed to attempting to interpret and organize unstructured text outputs. This structured approach is helpful for managing complex interactions, including tracking state, directing flow, and ensuring meaningful dialogue.

Implementing additional constraints through code within each node allows for precise control over the conversation. For instance, during a checkout process, code can verify credit card details against realistic parameters (e.g., validity, expiration date) that the LLM might overlook. Similarly, conversation depth and content within a node can be regulated—such as limiting the number of interactions in a greeting phase-or checking "facts" and sentiment coming from the LLM.

Overall, the prompt graph model enhances the conversation flow by combining the LLM's natural language capabilities with deterministic controls, thereby ensuring a coherent and contextually appropriate user experience.
An example: the GREETING NODE

### Welcome Node

Here's an example node form the pizza shop chatbot.

#### Node prompt

```md
The context for this node prompt is that the user will be placed into it when they first arrive, and also placed back here again if they start a new order within the same conversation.

Last Assistant Utterance: "{LAST_BOT_UTTERACE}"

Last User Utterance: "{LAST_USER_UTTERACE}"

{ADDITIONAL_ERROR_MESSAGES}

Based on the user's response to the greeting, the following information should be updated or maintained:

MEMORY_ITEMS: {MEMORY_ITEMS}

CURRENT_ORDER_PARTS: {CURRENT_ORDER_PARTS}

CURRENT_BASKET: {CURRENT_BASKET}

Based on the user's response to the greeting, determine the next step in the conversation, and reply to the user.

- If the user wants pizza, set the next node to "Choose Pizza Type" and remember their preference.

- If the user wants salad, set the next node to "Choose Salad Type" and remember their preference.

- If the user is unsure or asks for recommendations, provide a brief recommendation based on popular choices and ask their preference again, staying in the "Greeting" node.
```

Within the prompt, we're tracking all of the relevant items and order needed for a good customer experience. These are the "slots" from the graph based conversation. To make sure the LLM does not forget anything, we will create rules around how these items are allowed to change in between steps. For instance, you might enforce that CURRENT_ORDER_PARTS = None in the greeting node if the user is just starting an order.

In the case where any of the additional constraints are violated, we've got a template parameter ADDITIONAL_ERROR_MESSAGES to pass those as well to the LLM and generate the appropriate next steps. This allows it to more gracefully handle unexpected or out of sequence responses from the user.

At the end of the prompt the LLM is given a choice to remain on this node or move on to another node and begin an order (either salad or pizza).

#### Node JSON output

When the prompt is evaluated we get back structured output in the following schema.

```json
{
    "name": "process_greeting_node",
    "description": "Process user responses at the greeting node of a pizza and salad ordering chatbot conversation, navigating to the appropriate next node and updating order details.",
    "parameters": {
        "type": "object",
        "properties": {
            "assistant_utterance": {
                "type": "string",
                "description": "The message the assistant should say next, based on the logic described in the prompt."
            },
            "next_node": {
                "type": "string",
                "description": "The next node to navigate to based on user input ('Order Pizza', 'Order Salad', 'Greeting', 'Checkout')."
            },
            "memory_items": {
                "type": "object",
                "description": "Items to keep in memory for future decisions, including user's expressed preferences.",
                "properties": {
                    "user_preference": {
                        "type": "string",
                        "description": "The user's expressed preference (pizza, salad, or '') to be kept in memory for future decisions."
                    }
                }
            },
            "current_order_parts": {
                "type": "object",
                "description": "Information about the current item the user is ordering, initially set to empty or default values.",
                "properties": {
                    "item_type": {
                        "type": "string",
                        "description": "Updated based on the conversation flow, initially set to ''."
                    },
                    "size": {
                        "type": "string",
                        "description": "Updated based on the conversation flow, initially set to ''."
                    },
                    "extras": {
                        "type": "array",
                        "items": {
                            "type": "string"
                        },
                        "description": "Updated based on the conversation flow, initially set to an empty array."
                    }
                }
            },
            "current_basket": {
                "type": "array",
                "description": "All of the items currently in the basket for the customer, including item type, size, and extras.",
                "items": {
                    "type": "object",
                    "properties": {
                        "item_type": {
                            "type": "string",
                            "description": "Represents each item type the user has added to their basket."
                        },
                        "size": {
                            "type": "string",
                            "description": "Represents the size of the item if applicable."
                        },
                        "extras": {
                            "type": "array",
                            "items": {
                                "type": "string"
                            },
                            "description": "Represents any extras added to the item."
                        }
                    }
                }
            }
        }
    },
    "required": [
        "assistant_utterance",
        "next_node",
        "memory_items",
        "current_order_parts",
        "current_basket"
    ]
}
```

#### Additional constraints:

We can apply the constraints before or after evaluating the prompt and parsing the JSON output. Before we evaluate the prompt let's:
- Check if the user is a return customer and populate their preferences.
- Check if there are any time or geography based special offers we want to promote and populate them.
- If the user just arrived make sure the current_basket and current_order_parts are empty

After evaluating the last utterance and parsing the result:
- Check the user's language and update the preferences
- Check for abusive behavior by with the user or assistant (using a pre-trained machine learning model)
- Check to make that the next node is a valid option and there are no partial items in the current_order_parts

### Comparisons: Prompt Graph

The advantages of using a prompt graph over using a conventional graph based conversation are:

- Ease of composition You don't have to plan every single interaction. You're able to specify the conversation at a higher level than the utterance level and leave the details to the LLM.
- Simpler ML workflow You are not required to build datasets of intents and utterances. The LLM acts as the intent detection and dialogue generator for you.
- Conversation quality The conversations are more natural and more easily adaptable to new use cases.

The disadvantages are:

- Unexpected conversation flows Sometimes the LLM acts in a way that surprises and disappoints you. Then you have to add another rule to the prompt or to the constraints.
- LLM dependency The prompt graph is highly dependent on LLM quality. As the LLM changes over time, so too can the conversation quality.
- Price LLMs are expensive to operate.

The advantages of using a prompt graph over a single prompt chatbot are:

- More control You have more control over the general conversation flow and can apply different constraints in different contexts. This also allows improved error handling based on specific circumstances and context.
- Smaller prompts The prompts are smaller and more focused. That makes them easier to understand and update as well as cheaper and more performant to evaluate. This also allows them to scale to more complex conversations and workflows.
- Better auditability Analytics are easier - the context and intent of the conversation are segmented by which node they occurred in.
- Composability It's easier to make changes in one part of the conversation without altering behavior in another part. You can easily reuse nodes across different chatbots.

The disadvantages are:

- More complexity There are many more components to manage with a prompt graph. The software development and maintenance costs will be much larger.
- Less flexible You won't be able to order a pizza and a salad at the same time.

## Overall Comparison

### User Experience (UX)

#### Intuitiveness:

- Graph-Based: Can be less intuitive due to its structured nature, potentially making users feel restricted in their interactions.
- LLM-Based: More intuitive, as it allows for more natural language input from users, making the interaction feel more like a conversation with a human.
- Prompt Graph: Balances structured interactions with natural language input, aiming to enhance intuitiveness while maintaining a guided conversation flow.

#### Conversational Flow:

- Graph-Based: Might feel mechanical as the flow is predetermined by the graph structure.
- LLM-Based: Better at maintaining a natural and coherent conversation flow, adapting dynamically to user inputs.
- Prompt Graph: Leverages the strengths of both to provide a coherent conversation flow that can dynamically adapt to user inputs while following a logical structure.

#### Response Time:

- Graph-Based: Generally fast, as responses are pre-defined and triggered by specific intents.
- LLM-Based: Might be slower due to processing and generating responses on the fly.
- Prompt Graph: Aims to optimize response times by combining predefined responses for efficiency with the flexibility of on-the-fly generation when needed.

### Accuracy and Understanding

#### Language Processing:

- Graph-Based: Limited to recognizing predefined intents and may struggle with variations in language.
- LLM-Based: Superior, as it can understand a wide range of user inputs, including slang and typos.
- Prompt Graph: Enhances language processing capabilities by using LLM for understanding diverse inputs and graph-based logic to maintain accuracy in responses.

#### Error Handling and Context Handling:

- Graph-Based: May not handle errors or maintain context as gracefully, often requiring users to start over or follow specific prompts to correct misunderstandings.
- LLM-Based: Better at handling errors gracefully and maintaining context over a conversation, providing a more coherent interaction.
- Prompt Graph: Aims to offer robust error handling and context maintenance by utilizing LLM's ability to interpret varied inputs and graph-based structures to manage conversation flow and context.

### Scalability and Maintenance:

#### Graph-based Systems:

- Linear increase in complexity with additional nodes and intents.
- It can be challenging to update intents and nodes without disrupting existing flows.

#### LLM-based Systems:

- Large models can be expensive and slow, especially as the prompt and conversation grow.
- You must retrain to maintain performance, especially as language and user expectations evolve.
- Hard to interpret and debugging LLM decisions due to their "black box" nature.

#### Prompt Graph:

- The full conversation isn't included, and the prompt covers part not all of the entire task. This helps manage latency and costs.
- Potentially easier updates and maintenance due to localizing changes to specific nodes or intents. The node complexity can be adjusted (does it do a lot or a little?).

## Conclusion

In summary, our exploration into combining graph-based and LLM chatbots suggests a practical approach to improving chatbot design. The prompt graph method aims to balance the control and predictability of graph-based systems with the natural language capabilities of LLMs. The objective is straightforward: create chatbots that can handle complex interactions more naturally while remaining manageable for developers. This approach presents a viable solution to the current limitations of chatbot technologies, offering a pathway towards developing more efficient and user-friendly chatbots for a variety of applications. As we move forward, refining and applying this model will likely be a key focus for those in the field seeking to enhance chatbot functionality and user experience.


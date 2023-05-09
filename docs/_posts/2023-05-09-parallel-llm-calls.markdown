---
layout: post
title:  "Making Many LLM Calls: Prompt Execution"
date:   2023-05-08 00:00:00 -0500
categories: jekyll update
regenerate: true
---

## A Prompt Execution Engine

When managing a large number of LLM tasks, it's beneficial to execute them in parallel. Since this is a classic IO-bound process, I decided to use the [asyncio](https://docs.python.org/3/library/asyncio.html) library and leverage async/await.

Here's the execution block from a version of [The Big Lebowski Fact-Checker]({{site.url}}/jekyll/update/2023/05/08/chatbot-for-bi-walkthrough.html). In this example, the `doc_chunks` passed into the function are all strings with a certain number of tokens.

```python
async def get_document_facts(system_message: dict[str, str], user_facts: list[str], doc_chunks: list[str]) -> list[list[str]]:
    answer_refs = dict()
    n_chunks = len(doc_chunks)
    for chunk_id, doc_chunk in enumerate(doc_chunks):
        # Block 1: kick off all the work
        messages = list()
        doc_prompt = GET_DOCUMENT_FACTS_PROMPT.format(
            USER_FACTS=nl.join(user_facts), DOCUMENT_TEXT=doc_chunk
        )
        user_message = {"role": "user", "content": doc_prompt}
        messages.append(system_message)
        messages.append(user_message)
        answer_refs[chunk_id] = asyncio.create_task(get_reply(messages))
        await asyncio.sleep(0)

    answers = dict()
    timeout = 60
    start_time = time.time()
    while len(answers.keys()) < n_chunks and (time.time() - start_time) <= timeout:
        # Block 2: wait for the work to finish
        for task_name, task in answer_refs.items():
            if task.done() and task_name not in answers.keys():
                answers[task_name] = task.result()
        await asyncio.sleep(1)

    if len(answers.keys()) < n_chunks:
        logging.warn("timeout occurred")

    return_values = []
    for reply_msg in answers.values():
        # Block 3: parse the results
        reply_string = reply_msg["content"].replace("\\n", "\n")
        if (
            reply_string.find("documentFacts") >= 0
            and reply_string.find("NONE") <= 0
        ):
            fact_json = get_json(reply_string)
            documentFacts = fact_json.get("documentFacts", list())
            for x in documentFacts:
                return_values.append(x)
    return return_values
```

In `Block 1`, I iterate over all of the chunks that need summarization. I use the `asyncio.create_task` method to define the work and then `await asyncio.sleep(0)` to allow the task to start.

In `Block 2`, I wait for all tasks to complete. I set an arbitrary timeout for the longest running task of 60 seconds. The `await asyncio.sleep(1)` statement is inserted to prevent the loop from consuming all CPU cycles.

Finally, in `Block 3`, I parse the results from each task. Some tasks might return NONE instead of JSON (due to how I wrote the prompt), so I ignore those.

For tasks that require retrying or implementing fallback logic, I can modify the logic in `Block 2` to accommodate that.

## Conclusion

Efficiently managing multiple LLM tasks is crucial for improving the performance of your application. By using the asyncio library and async/await, you can execute tasks in parallel and handle them more effectively
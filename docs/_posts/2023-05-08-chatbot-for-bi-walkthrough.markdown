---
layout: post
title:  "Chatbot for BI walkthrough"
date:   2023-05-08 00:00:00 -0500
categories: jekyll update
regenerate: true
---

In this blog post I'm going to walk through the code for [baldrick](https://github.com/mcminis1/baldrick), a chatbot for BI.

Baldrick is a LLM powered chatbot that resides in slack, and queries bigquery for data. It was built from scratch in python Feburary of this year (2023) using GPT-3.5-turbo as its primary LLM backend. We chose to specialize it for the activity schema because I use it pretty regularly for user journey and product analytics at my day job (data science, analytics, engineering consulting).

## The User Experience

We always want to focus on the user experience when building applications. If it's not easy to use, and useful for solving a problem, then it's not worth building.

For this use case, the ideal user persona is a business user. This could be someone on the executive team, a sales person, bizdev, anyone who needs to use data to make decisions, but doesn't need to know SQL as a regular part of their job. They spend a lot of time using slack (or another team communication platform) and don't want to add another tool to their day.

Their ideal work flow would be to open a slack channel, ask a question (e.g. "How many widgets did we sell last month?" or "When was the last time someone from Acmesoft logged in?" or "Show me a weekly-cohorted retention analysis for my chatbot, Baldrick") and get back a correct answer and explanation of what that answer means.

|![](https://raw.githubusercontent.com/mcminis1/baldrick/main/img/slack_example_1_question.png)|
|:--:| 
| *Asking Baldrick a question* |

One concern is that this work flow can feel a bit like magic. If a business user asks a question and gets an answer, how do they know if it is right? You can't just surface the SQL query used to get their answer. Even if they know a little SQL, they may not know the data in the data warehouse well enough to know if every thing is right. Or, they may know where the data comes from and not know enough SQL to understand if the query is formed correctly.

The way we solved this is to take the query plan, explain to the user in natural language how it's going to work, then get the answer and show results inline. Explaining the query in natural language will not allow the user to catch all of the errors, but it should give them enough information to ask other folks and verify that the data sources are correct.

|![](https://raw.githubusercontent.com/mcminis1/baldrick/main/img/slack_example_1.png)|
|:--:| 
| *Baldrick giving you an answer* |

After displaying the answer and explanation, there's a few buttons used for feedback.

|![](https://raw.githubusercontent.com/mcminis1/baldrick/main/img/slack_example_1_interaction.png)|
|:--:| 
| *Baldrick getting feedback for improving the service (RFHL, and in-context learning)* |


Perhaps they know sql or they want to open a support ticket. They can 
view the SQL to copy/pase or inspect it.

They also have the option to thumbs up or thumbs down the results. This feedback is useful for channeling potential support issues to an analyst on the team and for improving the quality of the chatbot. I'll go into the feedback loop later in this post.

## Application Overview

The workflow has several steps:
1. A slash command in slack sends the user's request to the baldrick backend.
1. Baldrick acknowledges the command by repeating the query and reassuring the user that he has "a cunning plan".
1. Next we get all of the relevant activities for the business question. Note that in the activity schema, getting the activity is about the same as getting all of the relevant tables in "normal" data warehouse setups.
1. Next we get the quey to run. This involves giving the LLM the schema for each of the activities we've identified as being useful and some in-context learning. Using the new user question, we look up similar business question-SQL pairs that have been completed successfully in the past and include them as well in the prompt.
1. Next we see if the SQL is valid byt running the query through the planner. 
1. If the SQL fails the query planner, we take a single shot at trying to fix it. We pass the business question, the SQL, and the error to the LLM and ask it to fix the query.
1. We run the query and get the result.
1. We pass the business question, query, and result ot the LLM and ask for it to explain the whole things in terms a business user can understand.
1. Finally we pass the query, explanation, and results back to slack where it is formatted in a pretty text block for folks to see.
1. [Optional] The user can choose to view the query, and up or down vote the answer. These are implemented using buttons in the slack app. Responses are recorded along with the business question, sql, and some metadata in a database for future use in prompts and for RFHL learning.

The entire app is built on GCP and deployed to a google Cloud Run function. I set up a `workload_identity_provider` so that I could use a github action to securely push the code to google Cloud Build, store it on the Artifact Registry, and then deploy it to Cloud run.

## A Deep Dive Example

I want to do a deep dive into a single LLM prompt, how we generate the inputs, and how we parse the output. I'll use the one whose funciton is generating a SQL query using the activity schemas and some example queries ([github code](https://github.com/mcminis1/baldrick/blob/main/src/prompts.py#L102-L145)).

### The Prompt

Here is the prompt template:
```Given an input question, first create a syntactically correct BigQuery query to run. 
Only use the following table:
{table_info}
where the activity is one of these strings: {', '.join(self.activities)}
and the JSON schema for each of these activities is:
{self.valid_activity_schema}

Double check that your query obeys the following rules.
- You can order the results by a relevant column to return the most interesting examples in the database. 
- Unless the user specifies a specific number of examples, always limit your query to at most {top_k} results using the LIMIT clause.
- When performing aggregations like COUNT, SUM, MAX, MIN, or MEAN rename the column to something descriptive
- Use valid BigQuery syntax
- Use JSON_QUERY(feature_json, '$.json_path') to access fields in the activity JSON.
- Account for possible capitalization in in STRING values by casting them to lower case.
- All queries should be FROM {fully_qualified_table_name}
- If the user wants to limit the time range for the question use CAST(ts as DATE) and CURRENT_DATE() appropriately in the where clause.

Examples:
{ self.examples }

Question: {self.user_question}
BigQuery Statement:
```
We pass the prompt template all of the inputs and then get back the string for OpenAI to evaluate.


An example of the template with all of the fields filled in is:
```Given an input question, first create a syntactically correct BigQuery query to run. 
Only use the following table:
TABLE activities (
    activity_id     text,
    ts              timestamp,
    customer_id	    text,
    activity 	    text,
    anonymous_id    text,
    feature_json    object
)
where the activity is one of these strings: sale, pageview
and the JSON schema for each of these activities is:
ACTIVITY sale (
    url         text,
    item_name   text,
    item_price  decimal,
    quantity    int,
)
ACTIVITY pageview (
    url     text,
    manual  bool
)

Double check that your query obeys the following rules.
- You can order the results by a relevant column to return the most interesting examples in the database. 
- Unless the user specifies a specific number of examples, always limit your query to at most 5 results using the LIMIT clause.
- When performing aggregations like COUNT, SUM, MAX, MIN, or MEAN rename the column to something descriptive
- Use valid BigQuery syntax
- Use JSON_QUERY(feature_json, '$.json_path') to access fields in the activity JSON.
- Account for possible capitalization in in STRING values by casting them to lower case.
- All queries should be FROM ANALYTICS.ANALYTICS.ACTIVITIES
- If the user wants to limit the time range for the question use CAST(ts as DATE) and CURRENT_DATE() appropriately in the where clause.

Examples:
Question: How many widgets did we sell today
BigQuery Statement: select count(*) from ANALYTICS.ANALYTICS.ACTIVITIES where CAST(ts as DATE) = CURRENT_DATE()
and activity="sale"
and JSON_QUERY(feature_json, '$.item_name')="widget"

Question: How many pageviews did we get today
BigQuery Statement: select count(*) from ANALYTICS.ANALYTICS.ACTIVITIES where CAST(ts as DATE) = CURRENT_DATE()
and activity="pageview"

Question: How many sales per pageview have we gotten for the last 5 days?
BigQuery Statement:
```

There are a few things to notice here.
- The schema in the beginning is reinforced by the examples. So, the LLM has the chance to see the schema and query syntax a few times in the prompt. This seems to help it create better SQL. Without the in-context part, the LLM would generate errors at least 20% of the time. Adding in the examples made it much better.
- Over time We notices the same kinds of errors popping up. We had to extend the list of errors each time we saw a new one appear. For some of the trickier ones, we really could not get rid of them until we added in the examples.
- The examples should be formatted exactly the same way as the end of your prompt. This is the best way to get the LLM output to match what you want it to produce.
- This was created before OpenAI refactored the API to take in a list of messages. Today I'd put the first line into the `system` message along with a little bit more direction about how I wanted it to act.

### Embeddings and similarity

I've experimented with several different embedding libraries, both paid and free. I find that for this application, it made very little difference which one I chose (for other applications it can matter a lot). I chose "sentence-transformers/all-mpnet-base-v1" because it was reasonably fast and seemed to have good evaluation metrics.

The way the embeddings work:

1. Take the business question and pass it through the sentence_transformer. This gives you back a large vector.
1. Store the business question and SQL query in a dictionary where the key is the embedding vector.
1. When a user asks a new question, compute the embedding for that string.
1. Iterate through the items() in your dictionary and compute the inner product between the new vector and the key, that's the score.
1. Create a list of tuples where the first element is the score and the second is the dictionary value.
1. Sort that list in descending order of score and take the top K values.

And, in a nutshell, that's how a vector database works.

We can change how many examples we want to include. In my experience, a good rule of thumb is to have a few reinforced data points, but always fewer than the number of tokens the larges query might return.

## A List of Learnings

### LLM Application

- Latency is your #1 killer. Anytime new data is given to Baldrick, acknowledging it and giving the user a sense of progress is very important. A lot of times folks will abandon the query if it takes too long to compute.
- Parsing the outputs of prompts usually works. But, since it doesn't always work, you have to have fallback to repair broken data. I explored using the LLM itself as well as more deterministic ways and the jury is still out. Both work sometimes.
- Running out of tokens for your output results in broken output. It's important to count tokens and make sure you have enough left for your expected return value. I usually try to make sure the input takes less than 75% of my token count.

### Using a LLM for Analytics

- Using a LLM for SQL works really well. After implementing the in-context learning part I got basically zero errors for the SQL that got returned.
- Using the activity schema wasn't a terrible choice. There are no joins, so that's an advantage over other data warehouse strategies. However, it does have a different schema for each of the activity_json entities. SO, the complexity is still there. I imagine this would actually be increased over time since schema management in JSON is pretty poor (in my experience).
- Over time you're going to have problems migrating fine-tuned or in-context SQL to new schema. Managing that dataset will probably be harder than for dbt or other analytics frameworks since there could be a whole lot of different and weird queries in there.
- The state of synthetic data creation is sad. There are some tools like faker for making single fields. If you want to go out and create a synthetic dataset for a project like this, good luck! There aren't any tools I've found that make it easy to do that. If you do make somethign with a nice schema and rich set of events, it's not easy to put in real world correlations in it so that your results can be surprising or interesting.

### Deployment

- Setting up the workload identity provider took longer than it should. The documentation is not very up to date. After getting it set up, things worked smoothly and are presumably secure. It was a good learnign experience for me, but overkill for this size project.
- Cloud run is not great for this kind of thing. It takes a long time for the image to start up. Slack has a 3 second response limit. The result is that the user gets an error message at first, and then the messages gets retried and multiple jobs end up running.
- Pull your sentence_transformer model at build time. Then it'll be baked into the image. When the application starts it will just need to load the model (which is much faster than the initial pull).

## Conclusions

I built this project to see if anyone was interested enough to try to get it working, or modify it for their own use. I didn't get much interest from the community, but I did get some interest from other folks working in the space. Over the last few months, I've had 5-10 conversations with early stage technologists and founders about how to build stuff like this, what I found surprising, and where it all is going. 

It's an exciting time to be talking to folks and building new things. It seems like there are limitless possibilities. I hope this article spurred some new thoughts and ideas for your own projects!



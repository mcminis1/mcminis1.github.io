---
layout: post
title:  "LLM-powered Fact Checker walkthrough"
date:   2023-05-08 00:00:00 -0500
categories: jekyll update
regenerate: true
---

I recently build a LLM application I like to call [The Big Lebowski Fact Checker](https://yeahwellthatsjustlikeyouropinionman.com/). The code isn't public yet, but I'll include snippets here so you can understand how it works. If you go check it out, be patient please. Using LLMs isn't fast, and scaling is very slow for it on the google app engine.

## Motivation

I love building software with character. I think it adds an extra dimension to the application. In this case, I chose the big Lebowski. In the move there's a scene where The Dude says ["Yeah? Well, you know, that's just like uh, your opinion, man."](https://www.youtube.com/watch?v=j95kNwZw8YY) Well, that inspired me to search for the domain `yeahwellthatsjustlikeyouropinionman.com` which, you probably would not be surprised, was available!

Beyond the dope domain name, my motivation for building this app was threefold. The first is that there is a lot of press around how bad LLM hallucination is. LLMs are portrayed as very confident 8 year olds mansplaining to anyone who dares listen. I have seen this firsthand, but don't think it's essential. Including "facts" in the prompt and limiting the LLM to using data in the prompt to answer questions does a pretty good job eliminating make believe. It is certainly wrong in it's conclusion sometimes, but for that the prompt and data that goes into it shares the blame.

The second reason I built this is because it includes some essential pieces that are important for other apps I might want to build in the future. For instance, summarizing a web page, determining if a statement is supported by a text, and figuring out the right web searches to do to answer a question. Those are all parts I can pull out and use for other things.

The last consideration is where LLMs are headed. About 5 years ago there were large newsrooms where sports journalists wrote previews for sporting events. Now there are a small handful of companies that ingest statistics and produce previews for all of the major news outlets. Currently, fact checking is a manual process done in newsrooms by researchers. They have a process and are very thorough. I can imagine a future where they have a better suite of tools to use that allows them to be editors and proofreaders as opposed to search, summarize, and compare grunts.

## User Experience

The ideal user for this application is a fact checker. That's someone who has a statement that they need to corroborate or falsify.

When a user first visits they are greeted with a retro theme and text box to enter the statement they want to verify.

|![img]({{site.url}}/img/fact_checker/landing_page_with_fact.png)|
|:--:| 
| *Landing page where user has already entered a statement.* |

Once the user submits the statement, the backend starts working on verifying it. Since the latency is non-trivial, I provide a nice playlist of big Lebowski clips for your entertainment. 

|![img]({{site.url}}/img/fact_checker/landing_page_post_submit.png)|
|:--:| 
| *After submitting the fact you get a list of searches and sources.* |

Links to the sources are returned as well as the key text elements relevant to the statement.

|![img]({{site.url}}/img/fact_checker/landing_page_answer.png)|
|:--:| 
| *The sources are used to determine whether the statement is true, false, partially true, or unknown.* |

Finally, there is a verdict along with the supporting evidence.


## Application Overview

Here's the flow through the application
1. When the user lands on the page, open a websocket and start sending JSON across the wire.
1. The user submits their statement.
1. Lebowski responds by repeating the statement. This is just to make sure the statement stays in the frame because the text block toggles into the youtube player for in-flight entertainment.
1. Next we use the LLM to get a list of falsifyable statements included in the statement. If the LLM can't find any then the reply is "That's just like, your opinion man." Along with the statements, search terms are suggested for use in a google custom search engine.
1. Lebowski sends a message to the frontend with what statements it is going to try to verify along with the web searches.
1. The web searches are performed. For each search phrase we collect the top 3 hits from a list of several "trustworthy" sites.
1. For each of those search results we grab the html from the url and parse out the article text. 
1. Each paragraph of text from the webpage (new line separated) we compute it's embedding and insert it into a vector database.
1. Once we get all of the paragraphs inserted into the vector database, we embed the searches and the statements and retrieve the top 4 matches for each. These are then deduplicated and stored for use in the LLM later.
1. Lebowski sends the docs and links to the websites from the web search to the UI for your perusal man.
1. Finally we use the statements you want to verify and the information from the documents in a prompt to determine whether they are true or not, and which of the statements are most relevant for proving so.
1. That final message is sent to the frontend and the text box is reset.

## Information Extraction: Two Implementations

One of the steps in this process is extracting all information from a web page that's relevant to the user's statement.

If you randomly choose a webpage, there are probably ads, banners, links, and all kind of irrelevant lard in it. For instance check out [this Politifact page](https://www.politifact.com/factchecks/2022/dec/20/joe-biden/despite-his-claim-joe-biden-has-not-visited-afghan/). When you pull this up you'll see the header and article, but also all of the stuff towards the bottom including donation solicitation, sources, links to other fact checks, links to other parts of the site, copyright notices and other irrelevant stuff. 

As a human we can pretty quickly skim through the crap and find the important stuff. A web scraper can't do that. You might be able to create custom scrapers for every domain you want to scrape, but then you also have to keep them updated and build more new ones when you want to add new sources.

Here I'm going to try two different approaches to extracting the data: use a LLM to summarize and extract the relevant facts or compute embeddings to find the most relevant phrases. I tried the all LLM route first.

### LLM only: Summarization Challenge

The pure-LLM route involves getting the webpage html, parsing out the visible text, and passing the entire thing to the LLM in a prompt and asking it to extract the relevant parts.

For reference, this is how we get the visible text using BeautifulSoup:
```python
def get_text_from_url(url: str) -> str:
    s = requests.Session()
    s.headers.update(headers)
    response = s.get(url)

    soup = BeautifulSoup(response.content, "html.parser")
    for script in soup(["script", "style", "head", "title", "meta"]):
        script.extract()

    visible_text_elements = soup.find_all(text=True)
    visible_texts = []
    for element in visible_text_elements:
        e_str = (
            element.replace("\r", " ")
            .replace("\n", " ")
            .replace("\t", " ")
            .replace("\\r", " ")
            .replace("\\n", " ")
            .replace("\\t", " ")
            .strip(" \n\t")
        )
        if len(e_str) > 20:
            visible_texts.append(e_str)
    return visible_texts
```
This works reasonably well. The resulting array of text will contain some longer extraneous material but generally gets rid of a lot of noise. I'll discuss some further refinements that were necessary in the next section.

Once we have the text we want to parse we pass it to the method that extracts "facts" from the document. We're using GPT-3.5-turbo and splitting the messages into different roles. 

Here's an example prompt system message:
```python
GET_DOCUMENT_FACTS_SYSTEM = """You are a summarization and fact checking API written in typescript. Find supporting evidence that either proves or disprove the following statements. If the relevant evidence is in a table or chart include the data point but not the entire chart of table. Return valid JSON with a single 'documentFacts' key whose value is an array of strings. If the user statement is not supported or disproven by the document reply "NONE"."""
```

Here's the corresponding prompt
```python
GET_DOCUMENT_FACTS_PROMPT = """EXAMPLE
'''statements
dogs are animals
cats are dogs
fish and trees are animals
'''
'''document
encyclopedia definition of animals and the kingdoms
'''
ANSWER={{"documentFacts": ["mammals are in the animal kingdom","dogs and cats are mammals","fish are animals","trees are plants"]}}

EXAMPLE
'''statements
dogs are animals
cats are dogs
fish and trees are animals
'''
'''document
document about fried fish
'''
ANSWER=NONE

'''statements
{USER_FACTS}
'''
'''document
{DOCUMENT_TEXT}
'''
ANSWER="""
```

You might be able to guess the problem here. The page contents are just way too large to fit in the prompt without overflowing the context of the LLM (I do not have access to the 32K version of GPT-4).

So, the workaround I chose here is iterative summarization of chunks and then a deduplication step.

You can figure out the number of chunks you're going to need using [tiktoken](https://github.com/openai/openai-cookbook/blob/main/examples/How_to_count_tokens_with_tiktoken.ipynb) and some algebra:
```python
system_message = {"role": "system", "content": GET_DOCUMENT_FACTS_SYSTEM}
nl = "\n"
full_text_doc_facts_prompt = GET_DOCUMENT_FACTS_PROMPT.format(
    USER_FACTS=nl.join(user_facts), DOCUMENT_TEXT="\n".join(document_text)
)
user_message = {"role": "user", "content": full_text_doc_facts_prompt}

n_system_tokens = num_tokens_from_messages([system_message])
n_document_tokens = num_tokens_from_messages([system_message, user_message])
# we only want to pack the messages 3/4 full so that there is room for the reply tokens
per_msg_token_limit = 0.75 * MODEL_MAX_TOKENS
n_chunks = int(n_document_tokens / per_msg_token_limit) + 1
```

Now, this is really kind of an estimate because there's a fixed and a variable part to each message. The more accurate version computes the fixed parts of the prompt so you know how many actual tokens you can add to each chunk.

```python
no_text_doc_facts_prompt = GET_DOCUMENT_FACTS_PROMPT.format(
    USER_FACTS=nl.join(user_facts), DOCUMENT_TEXT=""
)
user_message = {"role": "user", "content": no_text_doc_facts_prompt}
base_document_tokens = num_tokens_from_messages([system_message, user_message])
allowed_tokens_per_partial_doc = per_msg_token_limit - base_document_tokens
doc_chunks = chunk_doc_list(document_text, allowed_tokens_per_partial_doc)
```

Doing it like that, you get an array of document chunks, and iterate over that to send them. In this prompt, I want to new line separate each of the paragraphs, so I include that in my chunker. Note that I could be overestimating the number of tokens in a given chunk if the '\n' is a part of a bpe after adding to the previous paragraph. This is close enough for me right now.

```python
nl_tokens = num_tokens_from_string("\n")
def chunk_doc_list(docs: list[str], max_tokens_per_chunk:int) -> list[str]:
    chunks = []
    chunk_str = ""
    chunk_tokens = 0
    for doc_part in docs:
        this_chunk_len = num_tokens_from_string(doc_part)
        if this_chunk_len + chunk_tokens <= max_tokens_per_chunk:
            chunk_str += "\n" + doc_part
            chunk_tokens += nl_tokens + this_chunk_len
        else:
            chunks.append(chunk_str)
            chunk_str = doc_part
            chunk_tokens = this_chunk_len
    chunks.append(chunk_str)
    for chunk in chunks:
        assert num_tokens_from_string(chunk) <= max_tokens_per_chunk
    return chunks
```

Doing it this way we split the doc into a bunch of chunks, summarize and extract the relevant info from each one and then re-combine them together. The recombined version can have the ame problem as these individual ones, so I deduplicate and summarize those as well it as well.

The system message and prompt go like this:
```python
DEDUPLICATE_FACTS_SYSTEM = """You are a deduplication API. Take the following list of facts are return a deduplicated and condensed list of facts. The resulting list should be shorter than the original list. Summarize the facts as much as possible. You must reply with valid JSON."""
DEDUPLICATE_FACTS_PROMPT = """EXAMPLE
'''facts
cats have 4 legs
dogs usually have four legs
cats and dogs are both animals with 4 legs
'''
RETURN={{"facts": ["cats and dogs usually have four legs and are animals"]}}

EXAMPLE
'''facts
spiders are hairy
dogs usually have four legs
cats and dogs are both animals with 4 legs
'''
RETURN={{"facts": ["spiders are hairy","cats and dogs are animals and usually have four legs"]}}

'''facts
{DOCUMENT_FACTS}
'''
RETURN="""
```

Similar to the last one, you have to split it up and do it iteratively to get a small number of facts that can fit into the next set of prompts. You can see that this can turn into a whole lot of LLM calls and a lot of added latency even if you parallelize them.

### LLM + Vector Database: Parsing Challenge

When using a [vector database](https://gist.github.com/mcminis1/2a2d639932b40d2571c06a3088b4c48c#file-vectordatabase-py) to find the relevant phrases, the html parsing and content extraction parts become much more important. The `sentence_transformers` models are much more sensitive to noise than the GPT model series.

Many webpages are off limits to web crawlers and robots. My approach to finding good reference material is probably a grey area, and so I'm not going to go into detail here. If you're really interested, try reaching out to me via linkedin or email.

The best library I've found for pulling text out of a website is [trafilatura](https://trafilatura.readthedocs.io/en/latest/)
```python
def get_text_from_html(contents_html: str) -> str:
    return trafilatura.extract(contents_html)
```
It's the only one I've seen with a really nice writeup on it's [evaluation metrics](https://trafilatura.readthedocs.io/en/latest/evaluation.html)

Here's how I filter out the top paragraphs.
```python
v = VectorDatabase()
def filter_texts(queries: list[str], full_texts: tuple[str, str]) -> dict[str,list[str]]:
    map_texts = {}
    for (link, full_text) in full_texts:
        texts = full_text.split('\n')
        map_texts[link] = texts
        v.add(texts)

    filtered_paragraphs = list()
    for query in queries:
        filtered_paragraphs += v.get(query, n=4)
    
    filtered_map = {}
    saved_paragraphs = set()
    for word in filtered_paragraphs:
        for key, val in map_texts.items():
            if word in val and word not in saved_paragraphs:
                saved_paragraphs.add(word)
                if key not in filtered_map:
                    filtered_map[key] = list()
                filtered_map[key].append(word)

    logging.debug(f"len filtered_paragraphs: {len(filtered_map)}")
    # clear all of these from the vector db.
    v.clear()

    return filtered_map
```
a few notes:
- I am choosing the top 4 documents for each query across all of the webpages I've scraped. I find that some of the web search results are not really relevant. 
- The queries are both the statements I'm trying to verify as well as the search terms I passed to google. I find that using both gives me more information to work with.
- If I were to revisit this I would probably keep the scores and try to do somethign clever with them to further reduce the number of paragraphs returned form the method.

### Comparison of Methods

In general, the LLM seemed better at identifying the right parts of information and extracting them. However, the cost was too high, both in terms of latency and price. Because it took so long to parse a single long page, I retrieved fewer results. This hurt my accuracy.

The vector database approach worked pretty well and shaved a significant amount of latency off the total execution time. One must be careful for the choice of embedding model, but once that is sorted, it does well enough. As you can see from the two sections above, the vector db approach is actually much simpler.

It's possible that the improved text extraction from the second approach would reduce the number of LLM calls in the first section. I didn't take the time to carefully check that.

LLM based extraction is abstractive and not extractive. So I can't point to a specific line in the original text where the information came from. LLM + vector DB is extractive, so, if I wanted to, I could construct a deep link back to the article and highlight the line it came from. I think both have their plusses and minuses, but for a fact-checking application, I think it's better to use direct references to the source material.

## GPT-3.5-turbo vs GPT-4

I developed both versions of the app using GPT-3.5-turbo. The day I got to my current stopping point, I gat API access to GPT-4. So, I thought I would try it out as well. My initial reaction to using it as a drop in replacement was "Wow, this is slow!" I've done enough performance tuning to get the relatively fast GPT-3.5 version down to around 20 seconds to complete. Using GPT-4 instead feels like I'm giving all that work back without much to show for it.

Without doing any additional prompt tuning, GPT-4 seemed to be about as accurate as GPT-3.5. In this case, I think it's because of the limitations of working with the in-context data. If I were to go full LLM end-to-end and use GPT-4, I would probably get better results, but the latency could easily reach into the minutes.

## Deployment

This time I deployed to google app engine. Using `gcloud app deploy` it was pretty much a breeze. There was a little bit of work that I had to do to get the custom DNS set up, and https working for both http and ws traffic, but wasn't too bad.

Here's the app.yaml for the app deployment.
```yaml

runtime: python
env: flex

instance_class: F2

entrypoint: uvicorn fact_checker.main:app --host 0.0.0.0 --port $PORT --workers 2

resources:
  cpu: 2
  memory_gb: 2.0
  disk_size_gb: 20

runtime_config:
  operating_system: "ubuntu22"
  runtime_version: "3.11"

includes:
  - env_variables.yaml
```
Here I had to split the yaml into 2 parts and add `env_variables.yaml` to `.gitignore` because it has my secret keys in it.

## Lessons Learned

Besides the LLM vs Vector database lessons above, there are a few other notes to make.

### Design

- I could have chosen a better theme/persona for the project. My goal was to build a real fact checker. Using a character from a famous movie who is kind of a screwball doesn't really match the tone of what I'm trying to build. The theme might make it easier for folks to want to take a look, but that's a casual interaction. After all, who really smiles when they click a link to see a app that just says true or false. One positive thing about the theme is that folks don't judge the output as harshly as if it were very polished and professional.

- Latency kills. I know this already from Baldrick, Chat GPT, and the other LLM apps I've built. But, this project required a ton of calls when using the LLM to summarize and extract info from web pages. When looking at the logs for the initial few users I shared it with, many folks never got to a conclusion. They thought it was done before it was even half way though. It took way too much time to get all of the data organized and useful. Adding in intermediate events and comments helped, but still wasn't enough.

- Front end isn't that bad. I used GPT to help with a lot of the CSS, HTML, and JS for the frontend. By the time I was done, I had a much better handle on it all. I think next time, I'll step up to using react instead.

- I'm starting to get enough reps at building these that I'm starting to feel the patterns I want to use. I'll write those up in more detail in a future blog post. In short they are around how to write classes that have a prompt and call the LLM, the execution of lots of async calls, and logging and feedback mechanisms.


## Conclusions

If you go check out the app, take it with a grain of salt. It's pretty under provisioned (this is not a paid service and I'm cheap). It should work reasonably well as a fact checking API, or at least be better than a coin flip.

I likely won't release the code for a while. It's quite a mess. Hopefully, you have a pretty good idea how it all fits together from the code snippets and outline I gave above. Don't hesitate to connect on linkedin to chat, share your experiences building, or pointing out a bug in my code. I hope you find something interesting and useful in it!
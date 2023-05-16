---
layout: post
title:  "2023-05-16 Weekly Update"
date:   2023-05-11 00:00:00 -0500
categories: jekyll update
regenerate: true
---

Here's my weekly roundup with a few interesting articles, an idea, and what I've been doing!

## Article Roundup

### [Brex's Engineering Guide](https://github.com/brexhq/prompt-engineering):

- This guide has a really good explanation from the history to advanced topics. I strongly suggest anyone working on prompts to give it a read.
- For me, the most interesting part was the [Command Grammars](https://github.com/brexhq/prompt-engineering#command-grammars) section. I haven't played around with these that much yet. I think there are some pretty interesting systems that could be built using them.

### [Transformers Agents](https://huggingface.co/docs/transformers/transformers_agents)

> In short, it provides a natural language API on top of transformers: we define a set of curated tools and design an agent to interpret natural language and to use these tools. It is extensible by design; we curated some relevant tools, but we’ll show you how the system can be extended easily to use any tool developed by the community.

- This allows python users to pass natural language instructions, images, and other assets to Huggingface where the data is processed and your desired results are returned. 
- It can [handle](https://huggingface.co/docs/transformers/transformers_agents#a-curated-set-of-tools):
    - Document question answering: given a document (such as a PDF) in image format, answer a question on this document (Donut)
    - Text question answering: given a long text and a question, answer the question in the text (Flan-T5)
    - Unconditional image captioning: Caption the image! (BLIP)
    - Image question answering: given an image, answer a question on this image (VILT)
    - Image segmentation: given an image and a prompt, output the segmentation mask of that prompt (CLIPSeg)
    - Speech to text: given an audio recording of a person talking, transcribe the speech into text (Whisper)
    - Text to speech: convert text to speech (SpeechT5)
    - Zero-shot text classification: given a text and a list of labels, identify to which label the text corresponds the most (BART)
    - Text summarization: summarize a long text in one or a few sentences (BART)
    - Translation: translate the text into a given language (NLLB)
- It's basically an AI personal assistant in your terminal or jupyter notebook.

### Anthropic: [Introducing 100K Context Windows](https://www.anthropic.com/index/100k-context-windows)

- This is a huge leap in context window size. For comparison, GPT-4 has a 8K window and 32K window version. However, the 32K version is in a very restricted beta phase. The only folks I know that have access to it are businesses.
- This will allow users to include an incredible amount of instructions and context for processing. For instance, you could put an entire small/medium sized code repo into it and get it refactored. You could put an entire book into it and ask questions over the entire text (book reports are so outdated).
- It _might_ be a death blow to vector databases. Vector databases have been very useful for including targeted context for LLM prompts. If you can just paste all the context you have, why filter it out using a vector database first?


## What I've been up to

### Explorations in Language: Verb Frames

Last week, I had a great discussion about a linguistic concept known as 'verb frames.' To put it simply, a verb frame provides a structured method to dissect a sentence into its basic components. This technique allows for a more profound understanding of language and sentence construction.

When we delve into a sentence using verb frames, we assign different roles to various parts of the sentence. These roles are referred to as 'Thematic Roles.' Each thematic role contributes to the overall meaning of the sentence. A comprehensive list of these roles and example classes that employ them can be found in [Table 2](https://verbs.colorado.edu/~mpalmer/projects/verbnet.html).

# A Verb Frame Example

Let's take a look at a simple sentence to illustrate the application of verb frames. Consider the sentence: "John gave Mary a book."

Here, "John" is the `Agent`, the one who performs the action. "Mary" is the `Recipient`, the one who receives something as a result of the action. "A book" is the `Theme`, the object that is transferred or moved around in the event.

So, if we decompose this sentence into a verb frame, we get:

`Agent V Recipient Theme`

Here, 'V' stands for the verb, which in our case is "gave". This simple representation allows us to understand the fundamental structure and roles within the sentence. By applying this frame to other sentences, we can quickly identify the sentence's key components and their respective roles.

You can find several examples [here](https://verbs.colorado.edu/verb-index/vn/hit-18.1.php).

# A Thought: Verb Frame and Sentence Embeddings

Sentence embeddings are mathematical representations of sentences. These representations are often created so that similar sentences map to similar points in a multi-dimensional space. This is essentially like translating the sentence from a linguistic form into a numerical form.

These numerical representations are often generated using machine learning techniques and are designed in such a way that they capture the semantics, or meaning, of the sentence. For example, the sentence "The cat sat on the mat" would have a different numerical representation from "The dog chased the ball", reflecting the different actions and actors in the sentences.

The advantage of these embeddings is that they allow computers to perform calculations on sentences. For example, you can measure the similarity between two sentences by calculating the distance between their embeddings. This is useful for many natural language processing tasks, such as information retrieval, semantic search, and machine translation.

With verb frames, we can provide structural information that further enriches the representation of sentences. By identifying the roles that different components of the sentence play, we might be able to create more nuanced and contextually aware sentence embeddings.

## Many Thanks

I want ot thank everyone that gave me feedback on the fact checker blog post. I have not incorporated it all yet, but have been slowly working through it.

## Sneak Peak: Knowledge Platform

|![Knowledge Platform Architecture](/img/updates/2023-05-16/knowledge_platform.png)|
|:--:| 
| *Knowledge Platform Architecture Diagram* |

I've also been working on building out a "Knowledge Platform" for a company I'm working with. They've got a collection of documents they want to enable semantic search and question and answering workflows on. It's an interesting problem space, with many products and companies popping up every week. There are many angles one might put on it as well (sales enablement, governance and compliance, enterprise search, self service BI, ...).

I'm in the early phases of the process and look forward to future updates ans the project progresses.


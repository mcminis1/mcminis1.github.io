---
layout: post
title:  "2023-05-23 Weekly Update"
date:   2023-05-22 00:00:00 -0500
categories: jekyll update
regenerate: true
---

[*Sign up for notifications and articles!*](https://buttondown.email/jmcminis)

Here's my weekly roundup with a few interesting articles, and what I've been doing!

## Article Roundup

# [LIMA: Less Is More for Alignment](https://arxiv.org/abs/2305.11206)

This is an academic article. I'll give you the executive summary. 

The punchline is from the Discussion:
```
We show that fine-tuning a strong pretrained language model on 1,000 carefully curated examples can
produce remarkable, competitive results on a wide range of prompts. However, there are limitations to
this approach. Primarily, the mental effort in constructing such examples is significant and difficult to
scale up.
```

### Unraveling the Research Paper

The research team meticulously assembled approximately 1000 high-quality prompt-response pairs. Here's an illustrative example:

```
[Advice (In Distribution)]
Q: My 6yo daughter is super smart and finds
the kids in school boring. How can I help
her make friends?

A: That’s great that your daughter is so smart!
It can be tough when kids are ahead of their
peers, especially socially. Here are a few
ideas for how to help her make friends:
• Talk with her about friendship. Ask her
what she thinks makes a good friend. Tell
her what you think. Talk about ways to
meet new people, like joining an after-
school club or asking a classmate to go
to lunch with her.

• Encourage her interests. Find out what
she likes and help her pursue those inter-
ests. If she loves animals, take her to the
zoo. If she enjoys building things, get
some Legos. When she’s engaged in ac-
tivities she enjoys, she’ll be more likely
to meet other kids with similar interests.

• Arrange play dates. [...]

• Teach her social skills. [...]

• Give her opportunities to practice. [...]
The most important thing you can do is let
her know that it’s okay to be different. Help
her embrace her uniqueness and celebrate
her strengths.
```

A significant portion, about 80%, of these pairs were drawn from three notable community Q&A platforms: Stack Exchange, wikiHow, and the Pushshift Reddit Dataset. The team handcrafted the remaining 20% to complete their collection.

The next step involved fine-tuning a Large Language Model (LLaMa-65B), which was subsequently subjected to a comprehensive human evaluation process.

[![Human evaluation of the Lima model](/img/updates/2023-05-23/lima_human_evaluation.png)](/img/updates/2023-05-23/lima_human_evaluation.png){:target="_blank"}

*In comparison to commercial and large open-source language models, Lima demonstrates commendable performance.*

### Discussion + Implications

This research paper is particularly intriguing as it demonstrates that just a few hundred to a thousand well-curated prompts can substantially enhance a large language model's performance, making it competitive with the best commercial offerings. Although the model's maximum potential is determined by the pre-training phase, an excellent user experience within a specific domain can be achieved with a set of well-crafted prompts. It dispenses with the need for person-years of reinforcement learning with human feedback in the loop, rendering it achievable for a small team committed to spending a week on prompt creation.

## [Neeva's Closure](https://neeva.com/blog/may-announcement)

Neeva, a well-funded startup supported by leading figures in the VC industry and founded by a team of ex-Googlers, recently closed its doors. Just a few months ago, they had launched an LLM-powered search engine.

Rumors suggest that Snowflake acquired Neeva, a development that I find particularly fascinating as it hints at a few possible futures.
1. Snowflake marketplace might now have an inbuilt search engine, simplifying the integration of external data into your data warehouse.
2. There could be an enhanced natural language interface for your queries.
3. Expect to see more LLM-based features throughout the Snowflake platform.

An official announcement is anticipated, which will shed light on how the amalgamation of a modern data warehouse and cutting-edge AI unfolds!

## [Microsoft Copilot 365](https://www.youtube.com/watch?v=S7xTBa93TX8)

Although it was introduced a couple of months ago, Microsoft Copilot 365 caught my attention recently. The product is an impressive assembly of prompts and use cases, seamlessly integrated into the Microsoft Office suite. As I'm not a Windows/Microsoft user, I'm not privy to its actual functionality or performance. Nevertheless, it's inspiring to see a myriad of ideas on enhancing productivity using LLMs, even if it's more aspirational than real at this point.

## Updates + Ideas

# Jamming on Artificial General Intelligence

Last week, I dedicated my time to an experiment centered around Artificial General Intelligence (AGI). My ambition was to develop a planning AI, capable of extending its own capabilities and devising work strategies for other agents to execute. While a week wasn't sufficient to fully realize my vision, I did make significant headway in crafting a functional prototype. 

Navigating the complexities of AI autonomy presents some notable challenges:
1. Enhancing instruction architecture: Crafting multi-step or hierarchical plans often leads to disappointing results. The need for better structured instructions is paramount.
2. Improving data collection: High-quality, comprehensive data is an essential yet elusive ingredient in our recipe for success.

Further insights about this fascinating journey in the world of AGI can be found in my recent blog post on the [current state of AGI]({% post_url 2023-05-21-adventures-in-agi-1 %}). 

# Using ChatGPT as a Student

A recent study that came to my attention indicated that one out of five college students incorporate GPT in their academic work; some submitting work generated by ChatGPT as their own. This is not only dishonest but also detrimental to their growth in writing skills. In light of that study, I have been contemplating how to guide my children in using ChatGPT as a writing assistant.

My personal usage of ChatGPT is as an editor to elevate the quality of my work. This, however, necessitates a clear understanding of what constitutes 'good' writing and a self-assessment of my strengths and weaknesses as a writer. Students beginning their writing journey may not be able to do that. As cognitive development progresses and writing skills are honed, we become better at distinguishing quality work and refining our own writing style. At this stage, using ChatGPT can be a significant aid, accelerating the writing and editing process, and allowing us to produce better work more swiftly.

To maximize the usefulness of AI assistants, students need to first understand the process and structure of writing. Armed with this knowledge, they can effectively use the AI assistant at each step, evaluate its suggestions, tailor them to their needs, and continually refine their work. I hope that curriculums can adapt to the technological landscape and incorporate these tools at the appropriate phases.

My advice for my your learners is informed by the AGI work I recently did. I want them to break the act of writing into phases and then work on each one. As they work through those phases, they can use ChatGPT as an editor, but not as a writer. I want them to form all of the ideas themselves, and let ChatGPT help them get rid of grammatical errors and tweak the language. When they are writing the final draft, they can read through what ChatGPT produces but have to completely rewrite it.

---
layout: post
title:  "Explorations in AGI"
date:   2023-05-21 00:00:00 -0500
categories: jekyll update
regenerate: true
---

[*Sign up for notifications and articles!*](https://buttondown.email/jmcminis)

For those unfamiliar with the term, AGI stands for [Artificial General Intelligence](https://en.wikipedia.org/wiki/Artificial_general_intelligence). While discussions on AI generally revolve around Narrow AI, which is designed to excel at a specific task (like self-driving cars), AGI stands a step ahead. Unlike Narrow AI that masters one task, AGI boasts a more versatile skill set, allowing it to adapt to a variety of situations by transferring its acquired knowledge across domains. Picture an AGI controlling your car; it wouldn't just navigate your route, but also, in case of a tire blowout, it would strategize the best course of action to get you back on track.

Today, the realm of AGI is more of uncharted territory, with ongoing experimental projects like Auto-GPT and BabyAGI trying to conquer it. 

[babyagi](https://github.com/yoheinakajima/babyagi): An AI-powered task management system
- [Architecture diagram](https://github.com/yoheinakajima/babyagi#how-it-works): The flow of information to complete tasks.
- [Follow on projects](https://github.com/yoheinakajima/babyagi/blob/main/docs/inspired-projects.md#inspired-projects): Many experiments came after this one in a multitude of languages and with a variety of tweaks to the experiment.

[Auto-GPT](https://news.agpt.co/): An experimental open-source attempt to make GPT-4 fully autonomous. More full featured than BabyAGI.
- [Plugins](https://github.com/Significant-Gravitas/Auto-GPT-Plugins#plugins): input and output.
- [Challenges](https://github.com/Significant-Gravitas/Nexus/wiki/Challenges): structured approaches to solving specific challenges.

Both BabyAGI and Auto-GPT work best using GPT-4 for planning and reasoning. No other large language models works as well. They utilize a sequence of prompts and memory storage to build, store, and process task lists. These platforms are quite user-friendly, and if you're familiar with launching a Docker container or using an IDE, you should feel at home. And if you're not a software development whiz, don't fret, as a hosted version is expected to roll out soon.

## Current performance

Despite the hype and excitement, my encounters with these frameworks have indicated there's lots of room for improvement. 

They fare well with straightforward tasks that can be accomplished by devising a sequential to-do list.
- Crafting a CSS class for a modern UI after researching 3 CSS frameworks.
- Scouting the 20 top jobs in AI companies for an administrative assistant role in San Francisco.
- Ordering a pepperoni pizza from the best pizza shop in Tampa for a pizza aficionado.

However, they falter when faced with dynamic or intricate tasks that require plan modifications based on newly acquired information or tasks with multiple interdependent phases. For example:
- Determining the most profitable software product to build.
- Designing a process for a general contractor to plan and execute a birdhouse building project with a team of subcontractors.

These frameworks often find themselves in an infinite loop due to unclear objectives or inadequate coding to complete a step, resulting in tasks being repeated incessantly. There are lots of examples of [not](https://www.reddit.com/r/AutoGPT/comments/12hqm7u/autogpt_issues_not_getting_any_to_completion/) [getting](https://github.com/Significant-Gravitas/Auto-GPT/issues/1994) [tasks](https://github.com/Significant-Gravitas/Auto-GPT/issues/2726) [to](https://github.com/Significant-Gravitas/Auto-GPT/discussions/2639) [complete](https://github.com/Significant-Gravitas/Auto-GPT/discussions/3962).

## Path Forward

Two crucial elements are being refined to overcome these challenges: 
1. More structured instructions.
2. Better, more comprehensive plugins.

# Structured Instructions

As the adage goes, "garbage in, garbage out," the same principle holds true for AI. Feeding the AI high-quality information is paramount for successful planning and task execution. 

While AI can code autonomously, its process might not mirror that of a seasoned developer. Let's consider a scenario where AI is tasked with selecting libraries and writing code to scrape websites. Ideally, if the AI had all the context, past experiences, and future plans that a senior developer has, it could potentially create something superior. But reality is different; AI only possesses the prompt's information and what it can find on the internet. This is often insufficient for flawless results. By providing comprehensive, high-quality information, we can help AI perform better.

A process, or a series of structured tasks, can enable the AI to approach senior developer-level proficiency. You can create synthetic personas (product manager, architect, senior developer, junior developer) and coordinate prompts that gradually plan the work, define the architecture, plan and outline development tasks, and finally, write and assemble the code. This approach aligns with [Chain-of-Thought (CoT) prompting](https://www.promptingguide.ai/techniques/cot), and significantly improves the chances of success for complex tasks.

For complex tasks, the probability of overall success decreases exponentially as the number of tasks increases ( $$ \( P_{success} \)^{n_{tasks}} $$). Hence, improving the success rate of each step is crucial. Structured tasks that can be composed aid in increasing the chances of success.


[Chain-of-Thought (CoT) prompting](). You can greatly improve the chances of success for complex processes by providing a structure to the AI on how to think about solving the problem.

# Plugins

At its core, AI is essentially a text API, receiving and returning text. Plugins serve as bridges, connecting the AI with real-world applications, data sources, and tools. Although AI can write its own code to expand its functionality, the complexity of these tasks could hamper your success rate. Preemptively planning how resources will be accessed and utilized can help streamline the process.

## Conclusions

Our journey towards AGI is still in its infancy. Although current tools like Auto-GPT and BabyAGI show promise, there's a long road ahead to achieve consistent usability. 

These early attempts serve as stepping stones, providing us with vital insights into the challenges that we need to overcome. The exploration of structured instructions and the development of more effective plugins hold immense potential in bringing us closer to the goal. 

As we delve deeper into the world of AGI, we can expect to witness many exciting breakthroughs. So, stay tuned as we perform experiments and continue to unravel the mysteries and opportunities of AGI.

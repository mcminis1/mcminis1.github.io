

## Lessons Learned

- A more suitable theme/persona for the project could have been chosen. The goal was to create a reliable fact-checker, but using a character from a well-known movie with a humorous tone may not have matched the intended purpose of the app. While the theme might have attracted users, it didn't necessarily encourage them to take the results seriously. On the other hand, the light-hearted theme did help users be less critical of the app's output.

- Latency is a major concern. Previous experiences with LLM apps already highlighted the importance of minimizing latency. In this project, however, the LLM's time-consuming process of summarizing and extracting information from web pages posed a significant challenge. Many users assumed the app had finished before it had even completed half of its tasks. Adding intermediate events and comments improved the situation, but more optimization is needed.

- Front-end development can be enjoyable. With the help of GPT, I learned to work more effectively with CSS, HTML, and JavaScript. This experience has encouraged me to explore using more advanced frameworks like React in future projects.

- Recognizing patterns is key. As I continued building LLM apps, they began to identify patterns in designing and structuring them. Some of these patterns involve creating classes with prompts and LLM calls, managing asynchronous calls, and implementing logging and feedback mechanisms. The author plans to share these insights in a future blog post.

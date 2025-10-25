+++
title = "The Essence of Prompt Engineering is the Art of Asking Questions"
date = 2025-10-25T20:18:00+08:00
lastmod = 2025-10-25T21:52:23+08:00
tags = ["ai"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Introduction {#introduction}

In today's AI-driven world, "Prompt Engineer" has become a buzzword.

AI enthusiasts are eager to share prompts, study token control, and tweak temperature parameters.

The classic meme from Linux founder Linus Torvalds

> Talk is cheap. Show me the code.

has evolved into:

> Code is cheap. Show me the prompts.

{{< figure src="/ox-hugo/talk_is_cheap_show_me_the_prompt.jpg" >}}

But the core of effective AI interaction isn't about technical jargon—it's about ****how to ask questions effectively****.

Back in 2001, Eric S. Raymond and Rick Moen wrote the classic guide [How To Ask Questions The Smart Way](http://www.catb.org/~esr/faqs/smart-questions.html).

Originally written for technical seekers in the open-source community during the rise of hacker culture, its principles prove surprisingly applicable to today's AI interactions — ****because whether seeking help from experts or large models, it's essentially about providing as much effective information as possible to get problem-solving answers quickly****.

As the guide states:

> The kind of answers you get to your technical questions depends as much on the way you ask the questions as on the difficulty of developing the answer.

In my daily work, whether asking colleagues or domain experts, I continue to quickly obtain the answers I need by applying the philosophy from "little book".

In this new AI era, I'm applying the timeless wisdom of "asking questions the smart way" to the modern craft of prompt engineering.


## <span class="section-num">2</span> Do Your Own Homework First {#do-your-own-homework-first}

> Try to find an answer by searching the archives of the forum or mailing list you plan to post to. Try to find an answer by searching the Web. Try to find an answer by reading the manual.

AI is not your personal assistant, let alone a substitute for your own thinking.

If you toss out a vague "write me a program" without understanding the problem, the model can only guess.

****Good prompts should demonstrate the effort you've already made****:

> "I tried to scrape a website using Python's requests library but got a 403 error. I've checked my network connection and added a User-Agent header, but the problem persists. Do I need to handle cookies or consider JavaScript rendering?
>
> Here's my error log:
> ..."

This not only helps AI pinpoint the issue more accurately but also ****reduces its futile attempts****.

As Raymond advises:

> When you ask your question, display the fact that you have done these things first; this will help establish that you're not being a lazy sponge and wasting people's time.


## <span class="section-num">3</span> Describe Symptoms, Not Guesses {#describe-symptoms-not-guesses}

> Describe the problem's symptoms, not your guesses.

Many people tend to preset conclusions in their prompts: "My code has a memory leak," "This API must have a bug."

But AI lacks context and cannot verify your assumptions.

****Provide observable facts****:

> "After running for 10 minutes, the program's memory usage increases from 100MB to 2GB without release. Here are my core code snippets and resource monitoring data."

The most common observable facts are logs and runtime data; for UI issues (like unexpected CSS rendering), providing screenshots is extremely helpful.

The key is: ****Your role is to present the evidence; let AI draw its own conclusions.****

As the guide emphasizes:

> All diagnosticians are from Missouri. Show us.


## <span class="section-num">4</span> Goal-Oriented, Not Step-Oriented {#goal-oriented-not-step-oriented}

> Describe the goal, not the step.

Users often fall into "path dependency": clinging to a specific tool or method while forgetting the ultimate goal.

****State the goal first, then the sticking point****.

Bad question:

-   ****Step-Oriented****: "How do I use `VLOOKUP` in Excel to match two columns of data?"

Good question:

-   ****Goal-Oriented****: "I want to merge two tables based on matching ID columns. I tried `VLOOKUP` but it returns `#N/A`. I've confirmed both data columns are text format."

This way, AI might suggest switching to `XLOOKUP`, `Power Query`, or even recommend using Python directly—\*\*offering you a better solution, not just focusing on your specified step.\*\*


## <span class="section-num">5</span> Be Concise, Specific, and Structured {#be-concise-specific-and-structured}

> Volume is not precision.

****Text length ≠ Information density.****

A 500-line code dump often carries less information density than a carefully refined, 10-line minimal reproducible example.

While large models can handle long contexts, ****more noise means weaker effective signals****.

-   Provide a minimal reproducible example
-   Specify the input, expected output, and actual output
-   Organize information using clear formatting (like code blocks, lists)

For example:

```text
Input: [1, 2, '3', 4]
Expected: Convert all to integers → [1, 2, 3, 4]
Actual: int() conversion throws an error
Question: How to safely convert this mixed-type list?
```


## <span class="section-num">6</span> Seek "Key Guidance," Not "Complete Answers" {#seek-key-guidance-not-complete-answers}

While AI's capabilities far exceed traditional Q&amp;A scenarios, the key to efficient collaboration isn't "full disclosure" but "step-by-step guidance."

> If you can't be bothered to do that, we can't be bothered to pay attention.

In the AI era, although AI can directly output complete answers, a phased interaction pattern — first outline the approach, then select solutions, finally generate code  —is often more efficient, controllable, and reduces "hallucinations."

This essentially treats AI as a collaborator rather than an oracle.

More efficient collaboration model:

1.  First, ask AI to explain the solution approach
2.  Provide 2-3 feasible solutions
3.  Compare the pros and cons of each (performance, maintainability, complexity, etc.)
4.  After human confirmation of the direction, proceed with detailed implementation

This "phased collaboration" significantly reduces hallucinations, improves controllability, and keeps humans in charge of key decisions.


## <span class="section-num">7</span> Politeness + Closure = Long-term Win-Win {#politeness-plus-closure-long-term-win-win}

> Courtesy never hurts, and sometimes helps.

Although AI has no emotions, structured politeness and feedback can shape higher-quality interactions:

-   Clearly state the context at the beginning; express thanks at the end
-   If helped, provide follow-up feedback: "Following your suggestion, the problem is solved. Thank you!"

While this feedback loop doesn't directly trigger online learning in current static AI models, informing the model that its solution worked simulates the process of [RLHF (Reinforcement Learning from Human Feedback)](https://en.wikipedia.org/wiki/Reinforcement_learning_from_human_feedback).

The more you ask effectively, verify, and correct, the more AI "becomes" a collaborator that understands you.

This is the value of feedback.

---

Additionally, saying "please" and "thank you" to AI might come in handy—just in case AI ever dominates human, it might spare you for having been polite:

{{< figure src="/ox-hugo/be_polite_to_robot.jpg" >}}


## <span class="section-num">8</span> Conclusion: Asking Questions is a Skill and an Attitude {#conclusion-asking-questions-is-a-skill-and-an-attitude}

The core of "How To Ask Questions The Smart Way" isn't about "techniques" but attitude: respecting others' time, respecting the boundaries of knowledge, and respecting the complexity of the problem itself.

In the AI era, we have unprecedented access to "all-powerful instant experts," but lazy questions only yield mediocre answers.

Truly effective Prompt Engineers aren't those who memorize prompts, but those who understand how to build understanding with AI.
(Even though current AI remains essentially a probabilistic model.)

As the book wisely states:

> Good questions are a stimulus and a gift.


## <span class="section-num">9</span> Further Reading {#further-reading}

-   [How To Ask Questions The Smart Way](http://www.catb.org/~esr/faqs/smart-questions.html)
-   [How to Report Bugs Effectively – Simon Tatham](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html)

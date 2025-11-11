+++
title = "Reflections on Ten Years of Programming"
date = 2024-12-15T21:09:00-08:00
lastmod = 2025-11-10T22:51:50-08:00
tags = ["summary", "life", "programming"]
categories = ["summary"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Preface {#preface}

Malcolm Gladwell's "10,000-hour rule" suggests that continuous investment of 10,000 hours of effort is sufficient to reach expert level in any field.
Based on 20 hours of practice per week, this requires about 3 hours of daily investment, taking roughly ten years to achieve this goal.

Since I wrote my first line of C code, more than ten years have passed.
During this period, I have written over 300,000 lines of code, some of which, written at WeChat, have served more than 1 billion users.

Despite having written so much code, I still dare not call myself an expert.

However, years of being a "builder", coding day after day, have allowed me to accumulate considerable insights.

"Practice makes perfect" - these insights are both reflections on programming technology and experiences of professional life.


## <span class="section-num">2</span> Continuous Learning {#continuous-learning}

Although I started programming with C in university, my main language in college was Java,
as Java is a very mature industrial language with rich frameworks, highly popular in enterprises, with a lot job opportunities.

I started web development with Java Servlets, then learned the very popular JavaEE enterprise development framework SSH, namely [Struts2](https://struts.apache.org/)&nbsp;[^fn:1]+ [Spring](https://spring.io/projects/spring-framework)&nbsp;[^fn:2]+ [Hibernate](https://hibernate.org/)&nbsp;[^fn:3]. Struts2 handled control logic, Spring provided decoupling, and Hibernate as ORM.

By the time I started job hunting, the SSH concept had changed - Struts2 was replaced by [SpringMVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)&nbsp;[^fn:4], and SSH became SpringMVC + Spring + Hibernate.

When I interned at Ant Group, I discovered that the team's codebase didn't use Hibernate for database, but rather [Ibatis](https://ibatis.apache.org/)&nbsp;[^fn:5], which later switched to the newer [MyBatis](https://mybatis.org/mybatis-3/)&nbsp;[^fn:6].

Ant Group internally didn't use Spring/SpringMVC either, but rather their in-house [SOFA framework](https://github.com/sofastack/sofa-rpc)&nbsp;[^fn:7]. The Spring community later felt that the Spring framework was too heavyweight and not conducive to rapid development, so they developed the lighter [SpringBoot](https://spring.io/projects/spring-boot)&nbsp;[^fn:8], while Ant internally launched the SOFA version called [SOFABoot](https://github.com/sofastack/sofa-boot)&nbsp;[^fn:9].

After joining WeChat Pay, I initially wrote C++ using WeChat in-house RPC framework called Svrkit. Later, due to Big Data projects, I started using Spark + Python + Hive SQL.

Now at AWS S3, because the business has extremely high performance and resource usage requirements, I'm using Rust again, while legacy business still uses Java. After going full circle, I'm back on the Java path.

Counting them all, over these years, I've written production code in Java, C++, Python, Rust, Scala, Kotlin, and JavaScript/TypeScript.

Beyond work, I've also learned Scheme while studying [SICP](https://github.com/ramsayleung/sicp_solution), Emacs Lisp from using Emacs, Swift for indie development, Ruby for Ruby on Rails, Golang for load testing, and various frameworks and libraries for different languages.

Since I started learning programming, I've learned at least half a dozen programming languages.

I never define myself as a programmer of any specific language, like Java programmer or C++ programmer - I just call myself a Software Development Engineer. Languages are merely tools; as long as I keep learning, I'll naturally learn new programming languages when encountering new scenarios.

The computer world changes rapidly - new frameworks might emerge every few months, and new languages become popular every few years. Continuous learning is essential for maintaining competitiveness..


## <span class="section-num">3</span> Independent Thinking {#independent-thinking}

WeChat used to have a tradition of giving out annual gifts to employees.

In 2022, the annual gift we received was indeed an aluminum plate with "Keep Independent Thinking 2022" written on it.

{{< figure src="/ox-hugo/think_independently.jpg" >}}

The creator of WeChat has always emphasized the importance of "independent thinking" for WeChat, believing that if he had to choose the most important quality, he would choose "independent thinking."

What superiors say isn't necessarily right, what teachers say isn't necessarily right, what academic institutions say isn't necessarily right, what media says isn't necessarily right, and loud voices certainly aren't necessarily right - after all, being reasonable doesn't require being loud.

For example, microservice architecture is very popular, and many companies are implementing microservices - does this mean monolithic architecture shouldn't be used?

Should startups or small teams adopt microservice architecture for new businesses? Or should they start with monolithic architecture and migrate to services as the business grows?

Development inevitably involves various decisions, such as technology selection. For your requirements, you might find a dozen components that "seemingly" meet the requirements,
and you might search online for evaluations of each component, finding diverse opinions. You need to independently analyze each component, identify their pros and cons, and make decisions based on your team's characteristics.

Regarding independent thinking, my favorite quote is from the HBO miniseries "Chernobyl,"
when scientist Valery Legasov asks the KGB to release his colleague Ulana Khomyuk, who was investigating the truth, saying he could guarantee she was no problem, the KGB chief responds:

> Trust, but verify.

Because the first step of independent thinking is: start questioning.


## <span class="section-num">4</span> Get It Running First {#get-it-running-first}

This phrase has a well-known variant: "It's not like it doesn't work."

Many programmers are perfectionists, especially those who have read "Refactoring" and "Design Patterns," who tend to spend a lot of time optimizing code and refactoring.

I used to have similar impulses, always wanting to spend time optimizing code, but after working on many projects, I have a strong feeling that it's better to get the MVP launch first and let users experience it early.

If no users are using it, even the best and most beautiful code is meaningless.

So when I often see community members asking what language and framework to use for side projects, whether PHP/Python/Ruby would be too slow, my view has always been: build a prototype first, find the first user first.

When performance becomes a bottleneck, your business is already very success, and you'll definitely have enough money to hire a dozen programmers to rewrite your project in Rust/C++, or assembly.

In this regard, I strongly agree with the my co-worker sitting next to me about code quality:

> make it run, make it fast, make it beautiful.

Recently attempting side projects, I have a profound realization that technology might be the least important aspect of business.

Building a product from scratch and promoting it to users - users only care whether your product is useful and can solve their problems.

They don't care whether you wrote it in C++/Java or JavaScript, nor do they care whether your code is elegant. Rather than obsessing over technology selection, it's better to build the product first and let users try it.


## <span class="section-num">5</span> The Most Comfortable Tool is the Best {#the-most-comfortable-tool-is-the-best}

I often see people in the community asking what's the best language, best framework, best editor, best operating system.

"Best" is quite a subjective conclusion, and there's no "best" solution for all scenarios, but I often see community members arguing over which language is better.

Or when someone shares about A, others reply that B/C/D is better, then arguments ensue.

This reminds me of the group identity phenomenon mentioned in "The Social Animal," a famous social psychology work.

When fans develop strong identification with a team, they view the team as part of their self-identity, leading them to:

1.  Use "we" instead of "they" to refer to the team
2.  View the team's success as personal success
3.  React defensively to criticism of the team, viewing such criticism as attacks on themselves

If someone asks me this question, I answer "the tool you're most comfortable and familiar with is best."

Even if for fun, programming's purpose is still to use computers to solve problems, and the best tool for solving problems is the one you're most familiar with.

Unless the tool you know isn't suitable for your problem, then naturally you need a new tool - don't force a square peg into a round hole.

Of course, if it's to satisfy curiosity and learn a new language, then choose what interests you.

When I learned Rust in 2017, it was simply because I had no classes in senior year, plenty of time, and wanted to learn something interesting and new. Rust 1.0 had only been released for 2 years then - I never expected to find work with Rust.

I can't remember where I read this passage:

> I once asked myself similar questions:
>
> 1.  Do good things necessarily become popular? Not necessarily
> 2.  Are things I like necessarily good things? Not necessarily
> 3.  Will I spend time and energy on something that might not become popular but I like? Yes


## <span class="section-num">6</span> Communicate More with People {#communicate-more-with-people}

While programmers certainly work with machines, fundamentally we're still solving human problems.

When I first learned programming, I had a misconception that as long as I mastered the technology, I wouldn't need to care about "interpersonal relationships."

Therefore, after entering the workplace, I held such views and acted accordingly. While I wasn't cold to others, I inevitably was, as friends described: "aloof."

But after working for a long time, I realized that "interpersonal relationships" are inevitable - which is called networking and connections.

Even if my technical abilities are solid, I need to be seen by others. Having good relationships with colleagues and leaders allows for "win-win" when achievements are made.

So now I usually chat with colleagues whether there's something specific or not, both to know each other better and learn more about the group, and to find potential optimization points from colleagues' complaints, practicing my philosophy of "Work hard and be nice to people."

After doing this job for a long time, I find out that software engineering is fundamentally human systems engineering.


## <span class="section-num">7</span> Code Isn't Everything {#code-isn-t-everything}

After writing programs for a while, it's easy to have illusion that everything can be solved with code.

When you're holding a hammer, everything looks like a nail.

The reality I learned is that many things cannot be solved with code. Code is just a tool that can only be used in appropriate scenarios - avoid path dependency.

Knowing how to write code isn't useful, you also need to write articles, give presentations within the company, let others "see you."

Professional ability in programming and project delivery is certainly important, but you also need soft skills in marketing yourself.

Chatting with the boss occasionally to increase communication and showing your visibility regularly might be more useful than launching ten projects.


## <span class="section-num">8</span> Work with Excellent People {#work-with-excellent-people}

After years in the industry, having worked at Ant Group, WeChat, and AWS, and having worked with all kinds of colleagues, I have an increasingly strong insight:

****Work with excellent people****

Not only can you learn excellent qualities from them and improve technical abilities, you can learn best practices and engineering experience. During code reviews, you can learn better programming approaches, and when encountering problems, you have reliable teammates to help and guide you.

You'll understand the uniqueness of systems developed by excellent programmers, know what simple and ergonomic systems look like, and develop your own technical taste.

Taste and aesthetics are abstract concepts, but after using good systems, you naturally won't be interested in those crude, poorly made systems that rely on boss endorsement for forced promotion.

Improving technical taste, while enhancing our technical cognition, can in turn help us improve design capabilities.

Another benefit of working with excellent colleagues is building high-quality professional networks, which benefits career development and provides more options when changing jobs or switching tracks.

Although startups also have excellent developers, on average, Big Tech have higher proportions of excellent programmers, as they offer more competitive salaries and benefits, naturally having higher recruitment standards.

For example, WeChat has a so-called interview committee(like a group of bar raiser). Besides interviewers from the hiring department, candidates must also pass interviews by committee interviewers to avoid lowering standards for quick hiring.

So I personally suggest that new graduates should go to big tech when possible, to gain experience.

Although I left WeChat almost two years ago, I still miss the colleagues I worked with in the same group. They were truly technically excellent, super nice people, and willing to help.


## <span class="section-num">9</span> Health Is the Foundation of Everything {#health-is-the-foundation-of-everything}

After programming for so many years, I've developed a pile of occupational diseases.

I had tenosynovitis since university, developed a lower back pain after working for a few years and my once thick, dark hair is now increasingly sparse.

Because Tencent headquarters had a free gym, I basically went to the company gym on workdays to take advantage of company benefits - 2 days of cardio running, 2 days of anaerobic equipment training, persisting for almost 3 years.

I also started paying attention to my diet, trying to eat less oil and sugar and avoid alcohol.

While fitness isn't a cure-all, at least I feel charged to handle high-intensity work.

Only when I lose something do I learn to cherish it. Only when I start taking medication and going to hospital for follow-ups do I start paying attention to my body.

Although programming is fun and supporting family is important, I still need to pay attention to my body. After all, health is the foundation of everything - if it breaks down, there are no other exciting stories.


## <span class="section-num">10</span> Summary {#summary}

Whether it's programming or other skills, I feel they all follow the "Matthew Effect" - the more I learn, the more I understand, and the faster you'll learn new things.

After writing a lot of code, I realized that a programmer's competitiveness isn't writing code, nor is it any particular language or framework.
The core competitiveness is the ability to solve problems through technology - why be constrained by any specific programming language or technology?

I hope ten years of programming is just a starting point, and in ten years I can write another piece: "Reflections on Twenty Years of Programming."

[^fn:1]: <https://struts.apache.org/>
[^fn:2]: <https://spring.io/projects/spring-framework>
[^fn:3]: <https://hibernate.org/>
[^fn:4]: <https://docs.spring.io/spring-framework/reference/web/webmvc.html>
[^fn:5]: <https://ibatis.apache.org/>
[^fn:6]: <https://mybatis.org/mybatis-3/>
[^fn:7]: <https://github.com/sofastack/sofa-rpc>
[^fn:8]: <https://spring.io/projects/spring-boot>
[^fn:9]: <https://github.com/sofastack/sofa-boot>

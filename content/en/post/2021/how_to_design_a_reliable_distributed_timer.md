+++
title = "How To Design A Reliable Distributed Timer"
date = 2021-08-05T09:19:36
lastmod = 2022-02-24T14:01:37+08:00
tags = ["distributed_system", "timer"]
categories = ["distributed_system"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Preface {#preface}

I have been maintained a legacy distributed timer for months for my employer, then some important pay business are leveraging on it, with 1 billion tasks handled every day and 20k tasks added per second at most.

Even though it's old and full of black magic code, but it also also have insighted and well-designed code. Based on this old, running timer, I summarize and extract as this article, and it wont include any running code(perhaps pseudocode, and a lot of figures, as an adage says: _A picture is worth a thousand words_).

if you are curious about the reason(I personally suggest to watch the TV series [Silicon Valley](https://www.imdb.com/title/tt2575988/), Richard has gave us a good example and answer)


## <span class="section-num">2</span> Design {#design}


### <span class="section-num">2.1</span> Algorithm {#algorithm}

There are several algorithms in the world to implement timer, such as Red-Black Tree, Min-Heap and timer wheel. The most efficient and used algorithm is timer wheel algorithm, and it's the algorithm we focus on.

As for timing wheel based timer, it can be modelled as two internal operations: _per-tick bookkeeping_ and _expiry processing_.

-   Per-tick bookkeeping: happens on every 'tick' of the timer clock. If the unit of granularity for setting timers is T units of time (e.g. 1 second), then per-tick bookkeeping will happen every T units of time. It checks whether any outstanding timers have expired, and if so it removes them and invokes expiry processing.
-   Expiry processing: is responsible for invoked the user-supplied callback (or other user requested action, depending on your model).


#### <span class="section-num">2.1.1</span> Simple Timing Wheels {#simple-timing-wheels}

The simple timing wheel keeps a large timing wheel, the below timing wheel has 8 slots, and each slot is holding the task which is going to be expired. Supposing every slot presentes one second(one tick as a second), then the current slot is `slot 1`, if we want to add a task needed to be triggered 2s later, then this task will be inserted into `slot 3`.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806103302.png" >}}

-   per-tick bookkeeping: O(1)

What happen if we want to add a task needed to be launched 20s later, the answer is we have no way to do so since there are only 8 slots. So if we have a large period of timer task, we have to maintain a large timing wheel with tons of slots, which requires exponential amount of memory.


#### <span class="section-num">2.1.2</span> Hashed Timing Wheel {#hashed-timing-wheel}

Hashed Timing Wheel is an improved simple timing wheel. As we mentioned before, it will consume large resources if timer period is comparatively large. Instead of using one slot per time unit, we could use a form of hashing instead. Construct a circular buffer with a fixed number of slots(such as 8 slots). If current slot is 0, we want store `3s later` task, we could insert into `slot 3`, then if we want bookkeep `9s-later` task, we could insert into `slot 1(9 % 8 = 1)`

-   per-tick bookkeeping: O(1) - O(N)

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806103527.png" >}}

It's a tradeoff strategy, We trade space with time.


#### <span class="section-num">2.1.3</span> Hierarchical Timing Wheels {#hierarchical-timing-wheels}

Since simple timing wheels and hashed timing wheel come with drawback of time efficiency or space efficiency. Back to 1987, after studying a number of different approaches for the efficient management of timers, Varghese and Lauck posted a paper to introduce [Hierarchical Timing Wheels](http://www.cs.columbia.edu/~nahum/w6998/papers/sosp87-timing-wheels.pdf)

Just make a long story short, I won't dive deep into hierarchical timing wheels, you could easily understand it by a real life reference: the old water meter

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210805205803.png" >}}

the firse level wheel(seconds wheel) rotates one loop, triggering the second level(minutes wheel) ticks one slot, same for the third level(hour wheel). Therefore, we present a day(60\*60\*24 seconds) with 60+60+24 slots. If we want to present a month, we only need to a four level wheel(month wheel) with 30 slots.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806103357.png" >}}

-   per-tick bookkeeping: O(1)


### <span class="section-num">2.2</span> Per-tick bookkeeping {#per-tick-bookkeeping}

After introducing timing wheel algorithm, let's go back to the topic about designing a reliable distributed timer, it's essential to decide how to store timer task. Taking implementation complexity and time, space trade off, we choose the Hashed Timing Wheel algorithm.

There are several internal components developed by my employer, one of them is named `TableKV`, a high-availability(99.999% ~ 99.9999%) NoSql service. `TableKV` supports 10m buckets(the terminology is `table`) at most, every `table` comes with full ACID properties of transactions support. You could simply replace `TableKV` with `Redis` as it provides the similar bucket functionality.


#### <span class="section-num">2.2.1</span> Insert task into slot {#insert-task-into-slot}

We are going to implement Hashed Timing Wheel algorithm with `TableKV`, supposing there are 10m buckets, and current time is `2021:08:05 11:17:33 +08=(the UNIX timestamp is =1628176653`), there is a  timer task which is going to be triggered 10s later with `start_time = 1628176653 + 10` (or 100000010s later, `start_time = 1628176653 + 10 + 100000000`), these tasks both will be stored into bucket `start_time % 100000000 = 28176663`

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806195209.png" >}}


#### <span class="section-num">2.2.2</span> Pull task out from slot {#pull-task-out-from-slot}

As clock tick-tacking to `2021:08:05 11:17:43 +08(1628176663)`, we need to pull tasks out from slot by calculating the bucket number: `current_timestamp(1628176663) % 100000000 = 28176663`. After locating the bucket number, we find all tasks in `bucket 28176663` with `start_time <` current_timestamp=, then we get all expected expiry tasks.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806195227.png" >}}


### <span class="section-num">2.3</span> Global clock and lock {#global-clock-and-lock}

As we mentioned before, when the clock tick-tacks to **current_time**, we fetch all expiry tasks. When our service is running on a distributed system, it's universal that we will have multiple hosts(physical machines or dockers), with multiple `current_times` on its machine. There is no guarantee that all clocks of multiple hosts synchronized by the same Network Time Server, then all clocks might be subtly different. Which current_time is correct?

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806105745.png" >}}

In order to get the correct time, it's necessary to maintain a monotonic global clock(Of course, it's not the only way to go, there are several ways to handle [time and order](http://book.mixu.net/distsys/time.html)). Since everything we care about clock is Unix timestamp, we could maintain a global system clock represented by Unix timestamp. All machines request the global clock every second to get the current time, fetching the expiry tasks later.

Well, are we done? Not yet, a new issue breaks into our design: if all machines can fetch the expiry tasks, these tasks will be processed more than one time, which will cause essential problems. We also need a mutex lock to guarantee only one machine can fetch the expiry task. You can implement both global clock and mutex lock by a magnificent strategy: an [Optimistic lock](https://en.wikipedia.org/wiki/Optimistic_locking)

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806114120.png" >}}

1.  All machines fetch global timestamp(timestamp A) with version
2.  All machines increase timestamp(timestamp B) and update version(optimistic locking), only one machine will success because of optimistic locking.
3.  Then the machine acquired mutex is authorized to fetch expiry tasks with timestamp A, the other machines failed to acquire mutex is suspended to wait for 1 seconds.
4.  Loop back to step 1 with timestamp B.

We could encapsulate the role who keep acquiring lock and fetch expiry data as an individual component named **scheduler**.


### <span class="section-num">2.4</span> Expiry processing {#expiry-processing}

Expiry processing is responsible for invoked the user-supplied callback or other user requested action. In distributed computing, it's common to execute a procedure by RPC(Remote Procedure Call). In our case, A RPC request is executed when timer task is expiry, from timer service to callback service. Thus, the caller(user) needs to explicitly tell the timer, which service should I execute with what kind of parameters data while the timer task is triggered.

We could pack and serialize this meta information and parameters data into binary data, and send it to the timer. When pulling data out from slot, the timer could reconstruct Request/Response/Client type and set it with user-defined data, the next step is a piece of cake, just executing it without saying.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806142855.png" >}}

Perhaps there are many expiry tasks needed to triggered, in order to handle as many tasks as possible, you could create a thread pool, process pool, coroutine pool to execute RPC concurrently.


### <span class="section-num">2.5</span> Decoupling {#decoupling}

Supposing the callback service needs tons of operation, it takes a hundred of millisecond. Even though you have created a thread/process/coroutine pool to handle the timer task, it will inevitably hang, resulting in the decrease of throughout.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806152334.png" >}}

As for this heavyweight processing case, Message Queue is a great answer. Message queues can significantly simplify coding of decoupled services, while improving performance, reliability and scalability. It's common to combine message queues with Pub/Sub messaging design pattern, timer could publish task data as message, and timer subscribes the same topic of message, using message queue as a buffer. Then in subscriber, the RPC client executes to request for callback service.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806161123.png" >}}

After introducing message queue, we could outline the state machine of timer task:

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806203653.png" >}}

Thanks to message queue, we are able to buffer, to retry or to batch work, and to smooth spiky workloads


### <span class="section-num">2.6</span> High availability guarantee {#high-availability-guarantee}


#### <span class="section-num">2.6.1</span> Missed expiry tasks {#missed-expiry-tasks}

A missed expiry of tasks may occur because of the scheduler process being shutdown or being crashed, or because of other unknown problems. One important job is how to locate these missed tasks and re-execute them. Since we are using global \`current_timestamp\` to fetch expiry data, we could have another scheduler to use \`delay_10min_timestamp\` to fetch missed expiry data.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806202724.png" >}}

In order to look for a needle in a haystack, we need to set a range(delay_10min - current time), and then to batch find cross buckets. After finding these missed tasks, the timer publishes them as a message to message queue. For other open source distributed timer projects like Quartz, which provides an instruction to handle missed(misfire) tasks: [Misfire instructions](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-04.html)

If your NoSql component doesn't support find-cross-buckets feature, you could also find every bucket in the range one by one.


#### <span class="section-num">2.6.2</span> Callback service error {#callback-service-error}

Since the distributed systems are shared-nothing systems, they communicate via message passing through a network(asynchronously or synchronously), but the network is unreliable. When invoking the user-supplied callback, the RPC request might fail if the network is cut off for a while or the callback service is temporarily down.

Retries are a technique that helps us deal with transient errors, i.e. errors that are temporary and are likely to disappear soon. Retries help us achieve resiliency by allowing the system to send a request repeatedly until it gets an explicit response(success or fail). By leveraging message queue, you obtain the ability for retrying for free. In the meanwhile, the timer could handle the user-requested retries: It's not the proper time to execute callback service, retry it later.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806195302.png" >}}


## <span class="section-num">3</span> Conclusion {#conclusion}

After a long way, we are finally here. The final full architecture would look like this:

The whole process:

1.  Adding a timer task, with specified meta info and task info
2.  Inserting task into bucket by hashed timing wheel algorithm(With `task_state` set to `pending`)
3.  Fetch_current scheduler tries to acquire lock and get global current time
4.  The Acquired lock scheduler fetches expiry tasks
5.  Return the expected data.
6.  &amp; 7. Publishing task data as message to MQ with thread pool; And then set `task_state` to `delivered`
7.  Message subscriber pulls message from MQ
8.  Sending RPC request to callback service(set `task_state` to `success` or `fail`)
9.  Retry(If necessary)

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210806202140.png" >}}

Wish you have fun and profit


## <span class="section-num">4</span> Reference {#reference}

-   [Paper: Hashed and Hierarchical Timing Wheels: Efficient Data Structures for Implementing a Timer Facility](http://www.cs.columbia.edu/~nahum/w6998/papers/ton97-timing-wheels.pdf)
-   [Hashed and Hierarchical Timing Wheels](https://blog.acolyer.org/2015/11/23/hashed-and-hierarchical-timing-wheels/)
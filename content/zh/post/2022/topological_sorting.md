+++
title = "Topological Sort"
date = 2022-05-22T10:34:00+08:00
lastmod = 2022-05-22T16:22:34+08:00
tags = ["graph", "algorithm"]
categories = ["algorithm"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Definition {#definition}

In computer science, a topological sort or topological ordering of a directed graph is a linear ordering of its vertices such that for every directed edge `uv` from vertex `u` to vertx `v`, `u` comes before `v` in the ordering.

It sounds pretty academic, but I am sure you are using topological sort unconsciously every single day.


## <span class="section-num">2</span> Application {#application}

Many real world situations can be modeled as a graph with directed edges where some events must occur before others. Then a topological sort gives an order in which to perform these events, for instance:


### <span class="section-num">2.1</span> College class prerequisites {#college-class-prerequisites}

You must take course `b` first if you want to take course `a`. For example, in your alma mater, the student must complete `PHYS:1511(College Physics)` or `PHYS:1611(Introductory Physics I)` before taking `College Physics II`.

The courses can be represented by vertices, and there is an edge from `College Physics` to `College Physics II` since `PHYS:1511` must be finished before `College Physics II` can be enrolled.

{{< figure src="/ox-hugo/course_prerequsites.png" link="/ox-hugo/course_prerequsites.png" >}}


### <span class="section-num">2.2</span> Job scheduling {#job-scheduling}

scheduling a sequence of jobs or tasks based on their dependencies. The jobs are represented by vertices, and there is an edge from `x` to `y` if job `x` must be completed before job `y` can be started.

{{< figure src="/ox-hugo/job_scheduling.png" link="/ox-hugo/job_scheduling.png" >}}

In the context of a CI/CD pipeline, the relationships between jobs can be represented by directed graph(specifically speaking, by directed acyclic graph). For example, in a CI pipeline, `build` job should be finished before start `test` job and `lint` job.


### <span class="section-num">2.3</span> Program build dependencies {#program-build-dependencies}

You want to figure out in which order you should compile all the program's dependencies so that you will never try and compile a dependency for which you haven't first built all of its dependencies.

A typical example is `GNU Make`: you specific your targets in a `makefile`, `Make` will parse `makefile`, and figure out which target should be built firstly. Supposing you have a `makefile` like this:

```makefile
# Makefile for analysis report

output/figure_1.png: data/input_file_1.csv scripts/generate_histogram.py
python scripts/generate_histogram.py -i data/input_file_1.csv -o output/figure_1.png

output/figure_2.png: data/input_file_2.csv scripts/generate_histogram.py
python scripts/generate_histogram.py -i data/input_file_2.csv -o output/figure_2.png

output/report.pdf: report/report.tex output/figure_1.png output/figure_2.png
cd report/ && pdflatex report.tex && mv report.pdf ../output/report.pdf
```

`Make` will generate a DAG internally to figure out which target should be executed firstly with typological sort:

{{< figure src="/ox-hugo/build_dependencies.png" link="/ox-hugo/build_dependencies.png" >}}


## <span class="section-num">3</span> Directed Acyclic Graph {#directed-acyclic-graph}

Back to the definition, we say that a topological ordering of a directed graph is a linear ordering of its vertices, but not all directed graphs have a topological ordering.

A topological ordering is possible if and only if the graph has no directed cycles, that is, if it's a directed acyclic graph(DAG).

Let us see some examples:

{{< figure src="/ox-hugo/directed_acyclic_graph.png" link="/ox-hugo/directed_acyclic_graph.png" >}}

The definition requires that only the directed acyclic graph has a topological ordering, but why? What happens if we are trying to find a topological ordering of a directed graph? Let's take the `figure 3` for an example.

{{< figure src="/ox-hugo/dag_issue.png" link="/ox-hugo/dag_issue.png" >}}

The directed graph problem has no solution, this is the reason why directed cycle is forbidden


## <span class="section-num">4</span> Kahn's Algorithm {#kahn-s-algorithm}

There are several [algorithms](https://en.wikipedia.org/wiki/Topological_sorting#Algorithms) for topological sorting, Kahn's algorithm is one of them, based on breadth first search.

The intuition behind Kahn's algorithm is pretty straightforward:

****To repeatedly remove nodes without any dependencies from the graph and add them to the topological ordering****

As nodes without dependencies are removed from the graph, the original nodes depend on the removed node should be free now.

We keep removing nodes without dependencies from the graph until all nodes are processed, or a cycle is detected.

The dependencies of one node are represented as in-degree of this node.

Let's take a quick example of how to find out a topological ordering of a given graph with Kahn's algorithm.

{{< figure src="/ox-hugo/kahn's_algorithm_1.png" link="/ox-hugo/kahn's_algorithm_1.png" >}}

{{< figure src="/ox-hugo/kahn's_algorithm_2.png" link="/ox-hugo/kahn's_algorithm_2.png" >}}

{{< figure src="/ox-hugo/kahn's_algorithm_3.png" link="/ox-hugo/kahn's_algorithm_3.png" >}}

{{< figure src="/ox-hugo/kahn's_algorithm_4.png" link="/ox-hugo/kahn's_algorithm_4.png" >}}

Now we should understand how Kahn's algorithm works. Let's have a look at a C++ implementation of Kahn's algorithm:

```C++
#include <deque>
#include <vector>
// Kahn's algorithm
// `adj` is a directed acyclic graph represented as an adjacency list.
std::vector<int>
findTopologicalOrder(const std::vector<std::vector<int>> &adj) {
  int n = adj.size();

  std::vector<int> in_degree(n, 0);

  for (int i = 0; i < n; i++) {
    for (const auto &to_vertex : adj[i]) {
      in_degree[to_vertex]++;
    }
  }

  // queue contains nodes with no incoming edges
  std::deque<int> queue;
  for (int i = 0; i < n; i++) {
    if (in_degree[i] == 0) {
      queue.push_back(i);
    }
  }

  std::vector<int> order(n, 0);

  int index = 0;
  while (queue.size() > 0) {
    int cur = queue.front();
    queue.pop_front();
    order[index++] = cur;

    for (const auto &next : adj[cur]) {
      if (--in_degree[next] == 0) {
	queue.push_back(next);
      }
    }
  }

  // there is no cycle
  if (n == index) {
    return order;
  } else {
    // return an empty list if there is a cycle
    return std::vector<int>{};
  }
}
```


## <span class="section-num">5</span> Bonus {#bonus}

When a pregnant woman takes calcium pills, she must make sure also that her diet is rich in vitamin D, since this vitamin makes the absorption of calcium possible.

After reading the demonstration of topological ordering, you (and I) too should take a certain vitamin, metaphorically speaking, to help you absorb. The vitamin D I pick for you (and myself) is two leetcode problems, which involve with the most typical use case of topological ordering -- college class prerequisites:

-   [Course Schedule](https://leetcode.com/problems/course-schedule/)
-   [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)


## <span class="section-num">6</span> Reference {#reference}

-   [Topological Sort | Kahn's Algorithm | Graph Theory](https://www.youtube.com/watch?v=cIBFEhD77b4)
-   [Directed Acyclic Graph](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/)
-   [Hands-on Tutorial on Make](https://gertjanvandenburg.com/files/talk/make.html)
-   [Topological sorting](https://en.wikipedia.org/wiki/Topological_sorting)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>


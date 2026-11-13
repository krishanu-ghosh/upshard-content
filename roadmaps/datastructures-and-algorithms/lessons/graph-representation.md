---
title: "Graph Representation"
description: Directed vs undirected graphs, adjacency lists vs adjacency matrices, and what a spanning tree is.
roadmap: datastructures-and-algorithms
node: graph-representation
group: core-data-structures
video:
  provider: youtube
  id: "gXgEDyodOJU"
  start: 0
  end: null
creator:
  name: mycodeschool
  channel_url: https://www.youtube.com/@mycodeschool
duration_seconds: 480
level: beginner
prerequisites:
  - trees-bsts-and-heaps
takeaways:
  - A graph is a set of vertices (nodes) connected by edges; a tree is really just a special case of a graph with no cycles and exactly one path between any two nodes.
  - In a directed graph an edge has a direction — A connects to B doesn't imply B connects to A — while in an undirected graph a connection goes both ways by definition.
  - An adjacency matrix stores an n-by-n grid of 1s and 0s and answers "are A and B connected?" in O(1), but costs O(n²) space even for a graph with very few edges.
  - An adjacency list stores, for each vertex, just the list of vertices it connects to, costing space proportional to the number of edges rather than the square of the number of vertices — the standard choice for sparse real-world graphs.
  - A spanning tree is a subset of a connected graph's edges that still touches every vertex but contains no cycles — the minimum set of connections needed to keep everything linked.
check_questions:
  - q: A social network where "A follows B" doesn't imply "B follows A" — is this best modeled as a directed or undirected graph, and why?
    a: Directed. The relationship is explicitly one-way — knowing A follows B tells you nothing about whether B follows A — so each edge needs a direction to capture that asymmetry correctly. A friendship graph where the relationship is always mutual would instead be undirected, since one edge naturally represents the connection both ways.
  - q: You're representing a social network of a billion users where the average person follows a few hundred others. Why would an adjacency matrix be a poor choice here even though it offers O(1) edge lookups?
    a: An adjacency matrix's size is fixed at n² regardless of how many edges actually exist, so a billion users would require roughly 10^18 cells — computationally and physically infeasible. Since each person only connects to a few hundred others, the graph is extremely sparse, and an adjacency list's space cost (proportional to the actual number of edges) would be many orders of magnitude smaller for the same data.
  - q: What's the difference between a graph and a tree, structurally?
    a: A tree is a connected graph with no cycles and exactly one path between any two nodes — every tree is a graph, but not every graph is a tree, since a general graph can have cycles and multiple paths between the same two nodes, or can be disconnected into separate pieces entirely.
related_formulas: []
---

## Graphs as the general case

Every tree seen so far in this roadmap — binary trees, BSTs, heaps — has been a graph with extra restrictions: no cycles, and exactly one path between any two nodes. A **graph** drops those restrictions and keeps just the core idea: a set of **vertices** (also called nodes) connected by **edges**. That generality is what makes graphs the right model for things that don't naturally form a hierarchy — road networks, social connections, dependency relationships, the web's hyperlink structure — where a node can connect to any number of others, cycles are completely normal, and there's often no single "root."

## Directed vs. undirected

The first choice in modeling something as a graph is whether its edges have a direction. In a **directed graph** (or digraph), an edge from A to B represents a one-way relationship — "A follows B on Twitter," "task A must finish before task B starts," "web page A links to page B." The reverse relationship, if it exists at all, is a separate edge from B to A. In an **undirected graph**, an edge is inherently symmetric — "A and B are Facebook friends," "city A and city B are connected by a two-way road" — a single edge between A and B fully describes the relationship in both directions. The same real-world structure can sometimes be modeled either way depending on what question you're trying to answer: a road network is usually undirected unless some roads are one-way, in which case those specific edges need direction while others don't.

## Two ways to store a graph

Once you know what the vertices and edges mean, you still need a concrete way to store them in memory, and the two standard choices trade space for lookup speed in opposite directions.

An **adjacency matrix** is an n-by-n grid, where n is the number of vertices. Cell (i, j) holds a 1 (or an edge weight) if there's an edge from vertex i to vertex j, and 0 otherwise. Checking whether two specific vertices are connected is then a single array lookup — O(1), as fast as it gets. The catch is space: the matrix always has n² cells, regardless of how many edges actually exist. For a **dense** graph — one where most pairs of vertices are connected — that's a reasonable price. But most real-world graphs are **sparse**: a social network with a billion users doesn't have anywhere close to a billion-squared friendships, and a road network doesn't have a direct road between every pair of cities. For a sparse graph, an adjacency matrix wastes enormous amounts of memory storing zeroes for pairs that were never going to be connected.

An **adjacency list** fixes that by storing, for each vertex, only the list of vertices it actually connects to — typically an array of lists (or lists of lists), one per vertex. Space usage is proportional to the actual number of edges, not the square of the number of vertices, which is why adjacency lists are the standard choice for the sparse graphs that dominate real applications. The tradeoff runs the other way for lookups: checking whether vertex A connects to vertex B now means scanning A's list of neighbors rather than a single array access, which costs time proportional to A's number of connections (its "degree") instead of O(1). For most graph algorithms — which typically need to visit "all of vertex A's neighbors," not just check one specific pair — that's exactly the access pattern an adjacency list is built for, which is part of why it's the default representation the traversal and shortest-path algorithms later in this roadmap are written against.

## Spanning trees

Given a connected undirected graph, a **spanning tree** is a subset of its edges that still touches every single vertex, contains no cycles, and therefore has exactly n − 1 edges for n vertices — the minimum number of connections needed to keep the whole graph linked together with no redundancy. A graph can have many different spanning trees (any edge involved in a cycle can usually be swapped for a different one without disconnecting anything), and if the graph's edges have weights, finding the spanning tree with the lowest total edge weight — the **minimum spanning tree** — is a well-studied problem with efficient algorithms (Prim's and Kruskal's, covered in the graph algorithms group next) with direct applications in network design: laying the least total cable to connect a set of buildings, for instance, is literally a minimum-spanning-tree problem.

## Setting up what's next

Representation is the foundation the rest of graph theory sits on: breadth-first and depth-first search, shortest-path algorithms, and minimum-spanning-tree algorithms are all written in terms of "visit a vertex's neighbors," which is precisely the operation an adjacency list is built to make convenient. Getting comfortable with directed-vs-undirected and matrix-vs-list now is what makes those algorithms read as straightforward refinements of a simple idea, rather than each one reinventing how to think about a graph from scratch.

# Dremel: Interactive Analysis of Web-Scale Datasets

## Introduction

Low cost storage has enabled to collection of vast amounts of data

Interactive analysis with low response times is helpful for analysts and engineers in tasks like:
- data exploration
- monitoring
- online customer support
- rapid prototyping
- debugging of data pipelines

Queries on large-scale datasets require parallelism of both storage and compute

Google uses clusters of commodity machines to parallelize
But such clusters are vulnerable to stragglers and failures - so fault-tolerance is essential

Dremel supports analysis of large datasets in situ
(Dremel is much faster than MapReduce)

Dremel query processing uses execution trees

Dremel uses a SQL-like language natively (doesnt covert to MapReduce)

Dremel uses a columnar storage representation which reduces storage and CPU cost thanks to compression

Dremel also makes novel use of columnar storage to represent nested data

## Background

A high-performance storage layer (such as GFS) is essential for in-situ data management
Enables fast data loading

## Data Model


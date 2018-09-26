---
layout: post
title:  "An embarrassingly parallel Big Data multiprocessing solution for Python"
date:   2018-08-10 12:00:32 +0200
categories: python
---

Ok, lets say we have a quite good machine, e.g. (in 2018) equipped
with a 16-core Threadripper and 128GB of RAM.  Let's also assume that
we have quite a big dataset in need to be processed in parallel on
this single machine.  We would like to have good (= high) CPU
utilization while being nice to memory.

Whats the "best" way to do this?

In the recent months, I have experimented a lot with Python's
multiprocessing capabilities, especially when dealing with large
amounts of data.  By large, I mean datasets that do not fit into main
memory.

Let's also assume for now, that we have one or more 

The solution I propose here is of course one of many possible
solutions, but---you surely heard such things before---it worked "best
for me".

I will show you how to solve this task using [pymp][pymp] for
multiprocessing and [PyTables][pytables] for data storage.

First of all, we need a nice dataset to operate on: Let's use the
[DEEP2 DR4 Redshift Catalog] data




to be continued...

[pymp]: https://github.com/classner/pymp
[pytables]: https://www.pytables.org/
[deep2]: http://deep.ps.uci.edu/DR4/zcatalog.html

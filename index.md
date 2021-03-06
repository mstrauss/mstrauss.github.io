---
layout: home
title:  "Regarding Python and 'Big Data'"
# permalink: python-123
date:   2019-05-13
---



<!-- ## General tips about using Python: -->

These are my own very personal and opinionated tips about how (not) to
work within the Python ecosystem.  I will update this page
continuously...

## Maintaining a sane (=controlled) software environment

* Use [pyenv](https://github.com/pyenv/pyenv) to install your latest
  and greatest Python while having full control over the installed
  Python packages (and most imporatantly their versions and
  dependencies).

* Use [Pipenv](https://pipenv.readthedocs.io/en/latest/) to manage
  ["virtual environments"][venv] and keep control of the package
  revisions and dependencies.  _Pipenv_ plays nice together with
  _pyenv_ and can also export _requirements.txt_ to be used e.g. in
  [binder](https://mybinder.org/).


## Persistence

* DO NOT use [Pickle][pickle] to _archive_ data or results.  Why?
  Because of this:

        UserWarning: Trying to unpickle YOUR VERY IMPORTANT DATA from
        version 0.19.1 when using version 0.19.2. This might lead to
        breaking code or invalid results. Use at your own risk.

    Instead, use [HDF5][hdf5] (e.g. via [PyTables][pytables]) or even
    better, go straight for [Postgres][postgres].


## Multiprocessing

* Go for [pymp][pymp] to get your code running [embarrassingly
  parallel][ep].

* [PyTables][pytables] supports (at least) parallel reading. See
  [here][pytables-parallel] for how it works.

<!-- * DO NOT use Python's -->
<!--   [warnings](https://docs.python.org/3/library/warnings.html) system -->
<!--   with _pymp_.  They do not like each other and child processes may -->
<!--   hang if they raise a warning, e.g. used together with -->
<!--   'simplefilter("error", ...)'. -->

* You have to take care to handle exceptions in child processes,
  otherwise they may hang.  See this [excellent
  post](https://stackoverflow.com/a/19929767/215431) for an overview.

  But it is easiest to not allow any exceptions"to spill out" from the
  subprocesses.  For example:

  ```python
  import traceback
  def subprocess():
    try:
      # do something
    except Exception as e:
      print("EXCEPTION:", e)
      traceback.print_exc()
    finally:
      # clean up
  ```

  If `pymp` does not work for you and/or you need more flexibility,
  the consumer-producer pattern may be for you: Use
  [multiprocessing][mp] directly like so:

  ```python
  import multiprocessing as mp
  import os
  from itertools import repeat
  from queue import Empty

  def compute(data_chunk):
      try:
          # do something useful with a chunk of data
          return result
      except Exception e:
          # report exception
          return None

  def producer_fun(job_queue, data_chunks):
      for chunk in data_chunks:
          job_queue.put(chunk)

  def consumer_fun(job_queue):
      while True:
          try:
              chunk = job_queue.get(timeout=5)
              r = compute(chunk)
              if r is None:
                  sleep(3)
                  # re-queue the failed job
                  job_queue.put(chunk)
              else:
                  # store result to disk

          except Empty:  # thrown by Queue#get after the timeout hits
              # we have an empty queue, therefore we exit this worker
              return None

  if __name__ == '__main__':

      # list of data chunks (use small things here, like a filename,
      # and load that file directly from the worker, `consumer_fun`)
      data = ...

      jobs = mp.Queue()  # our queue of data chunks

      # define the workers (consumers) and the producer
      consumers = list(
        repeat(mp.Process(target=consumer_fun, args=(jobs,)),
               times=os.cpu_count()))
      producer = Process(target=producer_fun, args=(jobs, data))

      # start the processes
      list(map(mp.Process.start, consumers))
      producer.start()

      # wait for all processes to complete
      jobs.close()
      jobs.join_thread()
      producer.join()
      list(map(mp.Process.join, consumers))

      # verify that you have all results
  ```

  The last step is important, because with the setup as presented
  here, it is *possible* that some jobs will not get completed.  You
  wanna at least know about it.


[mp]: https://docs.python.org/3.7/library/multiprocessing.html
[ep]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hdf5]: https://en.wikipedia.org/wiki/Hierarchical_Data_Format
[pickle]: https://docs.python.org/3/library/pickle.html
[postgres]: https://www.postgresql.org/
[pymp]: https://github.com/classner/pymp
[pytables]: https://www.pytables.org/
[pytables-parallel]: https://www.pytables.org/cookbook/threading.html
[venv]: https://docs.python.org/3/tutorial/venv.html

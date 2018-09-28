---
layout: home
title:  "Regarding Python and 'Big Data'"
# permalink: python-123
date:   2018-09-26
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
  revisions and dependencies.  _Pipenv_ plays nice together with _pyenv_.
  

## Persistence

* DO NOT use [Pickle][pickle] to _archive_ data or results.  Why?
  Because of this:

		UserWarning: Trying to unpickle YOUR VERY IMPORTANT DATA from
		version 0.19.1 when using version 0.19.2. This might lead to
		breaking code or invalid results. Use at your own risk.

	Instead, use [HDF5][hdf5] (e.g. via [PyTables][pytables]) or even
    better, go straight for [Postgres][postgres].


## Multiprocessing

* DO NOT use sklearn's joblib.  This library "has
  issues".  Instead go for [pymp][pymp] to get your
  code running [embarrassingly parallel][ep].
  
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


[ep]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hdf5]: https://en.wikipedia.org/wiki/Hierarchical_Data_Format
[pickle]: https://docs.python.org/3/library/pickle.html
[postgres]: https://www.postgresql.org/
[pymp]: https://github.com/classner/pymp
[pytables]: https://www.pytables.org/
[pytables-parallel]: https://www.pytables.org/cookbook/threading.html
[venv]: https://docs.python.org/3/tutorial/venv.html

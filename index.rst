.. write-docs documentation master file, created by
   sphinx-quickstart on Fri Apr 26 16:04:32 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to write-docs's documentation!
==================================


此项目用来维护大家学习的文档. 此文档支持 rst(**restructureText**) 和 md(**markdown**) 文件.

这份文档也是作为熟悉 restructureText, sphinx 和 readthedoc 的 demo.



.. toctree::
   :maxdepth: 2
   :caption: daily

   base/introduction.rst
   base/quickstart
   pipenv.md
   

.. _language-docs:

.. toctree::
   :maxdepth: 2
   :caption: language

   language/python原理.md
   language/python魔法方法.md


.. _data-docs:

.. toctree::
   :maxdepth: 2
   :glob:
   :caption: data

   data/pandas.md


.. _database-docs:

.. toctree::
   :maxdepth: 2
   :glob:
   :caption: database

   database/*


.. _data-structure-docs:

.. toctree::
   :maxdepth: 2
   :glob:
   :caption: data structure

   data_structure/*


.. _other-docs:

.. toctree::
   :maxdepth: 2
   :glob:
   :caption: other

   other/*
   concurrent/*



resources
    **readthedoc:** `https://readthedocs.org/`_

.. _https://readthedocs.org/: https://readthedocs.org/

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
.. * :ref:`modindex`


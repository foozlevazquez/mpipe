.. _concepts:

Pipeline concepts
=================

Pipelines are dynamic multi-processing workflows exhibiting some consistent patterns in their design:

#. a *worker* process operating on an input *task* and producing an output *result*,
#. a *stage* composed of one or more identical workers concurrently processing a stream of tasks,
#. a *pipeline* of stages linked together, output of the upstream stage fed as input into one or more downstream stages.

A *pipeline* is then conceptually composed of *stages*, each stage being an assembly of a number of (identical) *worker* processes. Messages that flow down the pipeline, from stage to stage, are called *tasks* or *results*, depending on whether they're viewed as input or output to a stage.

.. _task_result:

Task, result
------------

The unit of data that passes through the workflow is called a *task* when it is the input to a pipeline/stage/worker, and a *result* when it's the output thereof. 

.. image:: taskresult1.png
   :align: center

Most often a data object is both, depending on the context -- whether it's viewed as the output of an upstream producer, or input to the downstream consumer. It can be any Python object that can be dumped and read by the json.dump and json.reads methods: a standard Python data type like string or list, or a user-defined object. The only requirement is that it can be converted to json by the standard Python library.

Worker
------

.. image:: worker1.png
   :align: center

A worker is a basic unit of processing -- it operates some activity on a task, and (usually) produces a result. The worker is where you implement some elemental functionality of your overall program.

.. image:: worker2.png
   :align: center

A worker exists in a *stage*, alone or with other functionally identical workers, depending on how much system resources you choose to devote to that stage (each worker running in a separate process.) 

Stage
-----

The stage is an assembly of workers. It accepts a stream of input tasks, delegates them to its workers, and usually produces a stream of output results (products of its internal workers.) You can think of the stage as a corral of functionally identical workers, with added synchronization used to delegate tasks and organize results among the individual worker processes. The activity of the stage is therefore defined by the workers therein. 

A stage can be linked to another stage to form a chain:

.. image:: stage1.png
   :align: center

It can even be linked to multiple downstream stages, splitting the workflow into parallel execution paths.

Pipeline
--------

The pipeline is composed of linked stages forming a unidirectional workflow. 

.. image:: pipeline1.png
   :align: center

Input tasks are fed into the most-upstream stage. Pipeline results, if any, are fetched from outputs of downstream stages. 

.. _ordered_vs_unordered:

Ordered vs. unordered stages
----------------------------

This implementation considers all workers in a stage to be unordered, that is there is no synchronization between the workers in a given stage.

.. _multiprocessing:

Multiprocessing
---------------

The gist of structuring your program as a pipeline is to maximize algorithm speed by utilizing additional processing facilities of multi-core and multi-CPU computers. Parallel processing manifests itself in a number of ways in the pipeline workflow:

1. Pipeline can have multiple stages in series.

  The program is divided into a sequence of sub-algorithms. This benefits situations where the combined algorithm takes longer than the arrival interval between tasks. The pipeline can begin handling the next input task before the previous is completed.

2. Stage can contain multiple workers.

  If the input stream is even faster, such that computing time for a stage is longer that the interval between incoming tasks, additional workers at the stage can ameliorate a bottlenecked flow. Take a look at :ref:`multiple_workers_per_stage` for an illustration of potential speedup using this strategy.

3. Pipeline can fork into parallel stages.

  If the program workflow can be split into multiple independent execution paths, then parallel paths can be processed simultaneously.

.. End of file.

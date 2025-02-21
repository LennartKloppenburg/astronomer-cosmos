Render Config
================


Cosmos aims to give you control over how your dbt project is rendered as an Airflow DAG or Task Group.
It does this by exposing a ``cosmos.config.RenderConfig`` class that you can use to configure how your DAGs are rendered.

The ``RenderConfig`` class takes the following arguments:

- ``emit_datasets``: whether or not to emit Airflow datasets to be used for data-aware scheduling. Defaults to True
- ``test_behavior``: how to run tests. Defaults to running a model's tests immediately after the model is run. For more information, see the `Testing Behavior <testing-behavior.html>`_ section.
- ``load_method``: how to load your dbt project. See `Parsing Methods <parsing-methods.html>`_ for more information.
- ``select`` and ``exclude``: which models to include or exclude from your DAGs. See `Selecting & Excluding <selecting-excluding.html>`_ for more information.
- ``selector``: (new in v1.3) name of a dbt YAML selector to use for DAG parsing. Only supported when using ``load_method=LoadMode.DBT_LS``. See `Selecting & Excluding <selecting-excluding.html>`_ for more information.
- ``dbt_deps``: A Boolean to run dbt deps when using dbt ls for dag parsing. Default True
- ``node_converters``: a dictionary mapping a ``DbtResourceType`` into a callable. Users can control how to render dbt nodes in Airflow. Only supported when using ``load_method=LoadMode.DBT_MANIFEST`` or ``LoadMode.DBT_LS``. Find more information below.
- ``dbt_executable_path``: The path to the dbt executable for dag generation. Defaults to dbt if available on the path.
- ``env_vars``: (available in v1.2.5, use``ProjectConfig.env_vars`` for v1.3.0 onwards) A dictionary of environment variables for rendering. Only supported when using ``load_method=LoadMode.DBT_LS``.
- ``dbt_project_path``: Configures the DBT project location accessible on their airflow controller for DAG rendering - Required when using ``load_method=LoadMode.DBT_LS`` or ``load_method=LoadMode.CUSTOM``

Customizing how nodes are rendered (experimental)
-------------------------------------------------

There are circumstances when choosing specific Airflow operators to represent a dbt node is helpful.
An example could be to use an S3 sensor to represent dbt sources or to create custom operators to handle exposures.
Your pipeline may even have specific node types not part of the standard dbt definitions.

The following example illustrates how it is possible to tell Cosmos how to convert two different types of nodes (``source`` and ``exposure``) into Airflow:

.. literalinclude::  ../../dev/dags/example_cosmos_sources.py
    :language: python
    :start-after: [START custom_dbt_nodes]
    :end-before: [END custom_dbt_nodes]

When defining the mapping for a new type that is not part of Cosmos' ``DbtResourceType`` enumeration, users should use the syntax ``DbtResourceType("new-node-type")`` as opposed to ``DbtResourceType.EXISTING_TYPE``. It will dynamically add the new type to the enumeration ``DbtResourceType`` so that Cosmos can parse these dbt nodes and convert them into the Airflow DAG.

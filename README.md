# Airflow connector for Ocean Apache Spark

An Airflow plugin and provider to launch and monitor Spark
applications on the [Ocean for
Spark](https://spot.io/products/ocean-apache-spark/).

## Compatibility

`ocean-spark-airflow-provider` is compatible with both Airflow 1 and
Airflow 2. it is detected as an Airflow plugin by Airflow 1 and up,
and as a provider by Airflow 2.


## Installation

TODO(crezvoy): setup pypi repo
```
pip install ocean-spark-airflow-provider
```

## Usage

For general usage of Ocean for Spark, refer to the [official
documentation](https://docs.spot.io/ocean-spark/getting-started/?id=get-started-with-ocean-for-apache-spark).

### Setting up the connection

In the connection menu, register a new connection of type **Ocean For
Spark**. The default connection name is `ocean_spark_default`. You will
need to have:

 - The Ocean Spark cluster ID of the cluster you just created (of the
   format `osc-e4089a00`). You can find this in the console in the
   [list of
   clusters](https://docs.spot.io/ocean-spark/product-tour/manage-clusters),
   or by using the [Get Cluster
   List](https://docs.spot.io/api/#operation/OceanSparkClusterList) in
   the API.
 - [A Spot
   token](https://docs.spot.io/administration/api/create-api-token?id=create-an-api-token)
   to interact with Spot API.
 
![connection setup dialog](./images/connection_setup.png) 

The **Ocean For Spark** connection type is not available for Airflow
1, instead create an **HTTP** connection and fill your cluster id as
**host** your API token as **password**.

### Using the operator

```python
from airflow import __version__ as airflow_version
if airflow_version.starts_with("1."):
    # Airflow 1, import as plugin
    from airflow.operators.ocean_spark import OceanSparkOperator
else:
    # Airflow 2
    from ocean_spark.operators import OceanSparkOperator
    
# DAG creation ...
    
spark_pi_task = OceanSparkOperator(
    job_id="spark-pi",
    task_id="compute-pi",
    dag=dag,
    config_overrides={
        "type": "Scala",
        "sparkVersion": "3.2.0",
        "interactive": False,
        "image": "gcr.io/datamechanics/spark:platform-3.2-latest",
        "imagePullPolicy": "IfNotPresent",
        "mainClass": "org.apache.spark.examples.SparkPi",
        "mainApplicationFile": "local:///opt/spark/examples/jars/examples.jar",
        "arguments": ["100000"],
        "driver": {
            "cores": 1,
            "coreLimit": "1200m",
            "memory": "1g",
            "labels": {"version": "3.0.0"},
        },
        "executor": {
            "cores": 2,
            "memory": "1g",
            "labels": {"version": "3.0.0"},
            "instances": 2,
        },
    },
)
```

## Test locally

You can test the plugin locally using the docker compose setup in this
repositor. Run `make serve_airflow2` at the root of the repository to
launch an instance of Airflow 2 with the provider already installed.

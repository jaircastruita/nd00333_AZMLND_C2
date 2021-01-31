# Creating, deploying, consuming and pipelining an automl model from azureML
<br>

Using all the tools offered with azure ML so far the objective for this project is create, deploy and consume an auto machine learning classification project for the data banking dataset.

## Architectural Diagram
<br>

![azureml flowchart](images/azureml_flowchart.png)

Using automated ML to explore best model and hyperparameters: Instead of permuting different model options with their respective hyperparameters we let the automated machine learning service to do the heavy lifting for us. This process uses successful techniques used in order to explore the different models and hyperparameter space to select the best performing combination according to one given metric to optimize.

Deploying the best performing model found as a service: Once the best model is found sometimes is not enough. The main reason to do such exploration is to to use its predictions as a service. To achieve this we deploy the best performing model creating an endpoint. From there the model can be accessed through an API RESTful service (Asuming that user has the required credentials to call such service).

Enabling deploy logging: after deploying our model it is time to enable logging capability to generate text registries of performance for monitoring and comparison.

Creating swagger documentation: Using the built-in swagger documentation tool helps to speed up the adoption of deployed models through API services HTTP requests specifying the input arguments that de model accepts.

Creating a pipeline from a deployed model: ML pipelines stitches together different phases of ML. And it's mainly used to monitor performance in the real world, logging metrics and models outputs and also helps for detecting data drift and give the opportunity to reuse computed modules if changes weren't done.


## Key Steps 
<br>

In this section it is listed the steps followed in order to finish this project. There is a screenshot attached to each required step (Some screenshots were taken in two different sessions so there might be some difference in instances naming).

### Loading a dataset to azure ML

Fist, load the dataset which will be used to predict using a ML model. For this project the [bank marketing](https://archive.ics.uci.edu/ml/datasets/bank+marketing) dataset will be loaded. Direct to 'dataset' section, then, 'create new dataset'. You'll be requested to fill some properties about the dataset like 'name' and type of the dataset (in this case we are using a Tabular dataset). You can also use the [SDK method](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-create-register-datasets#create-a-tabulardataset) if you prefer using a python notebook. When you finish this process, a new dataset item will be listed with the name given in the 'dataset' section:

![registered dataset](images/proyecto2/04_registered_dataset.PNG)

### Finding best performance model with AutoML

Once the dataset is uploaded, it will be necessary to [create a running automl instance](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-use-automated-ml-for-ml-models#create-and-run-experiment) to perform the model/hyperparameter search. In your azure ML portal, go to the 'Automated ML' section and select 'New automated ML run' to start a new automl instance. You'll be requested to populate a form for basic information, it is also needed to select a datastore that you want to use for this experiment. You need to select the target column that will be used to fit the automl instance, and check the type assigned (automatically) to each column. You also will need to give a compute instance that will be used to run the experiment, if you not have one you will need to [create is as well](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-create-attach-compute-studio#compute-instance). The 'task type' is needed also. This specifies what your automl is being optimized. For this dataset you need to select 'classification' task. Another necessary requirement is to give a 'primary metric' to optimize. In this project 'Accuracy' (the default) is enough. For this experiment it is needed to cap the max duration to 1hr (3hr is the default), for timing restrictions.

Once all the necessary requirements are filled you can [run your experiment](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-use-automated-ml-for-ml-models#create-and-run-experiment). A new 'running' will appear on 'status', indicating that the experiment is running. Once the experiment is finished this label will be marked as 'completed'. 

![best model](images/proyecto2_2/s2_best_model.PNG)

The process is going to take some while but once it's done you'll be left with a completed automl project with several different models with preprocessing techniques and several metric plots to inspect and compare each model's performance [that you can access](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-use-automated-ml-for-ml-models#view-training-run-details).

![completed experiment](images/proyecto2_2/s2_experiment_completed.PNG)

### Deploying a model

Once the experiment is completed you can select the best performing model according to your selected 'primary metric' and [deploy it](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-use-automated-ml-for-ml-models#deploy-your-model).

#### Enabling Application insights

When you deploy a fitted model in a real environment it's possible that your model stumbles with some anomaly that you didn't forsee. In that case it is wise to prepare a series of pre-built modules offered by azureML to log any exception, error or just to register metrics that your deployment outputs.

We choose the best model for deployment and enable "Authentication" while deploying the model using Azure Container Instance (ACI). The executed code stored in logs.py [enables Application Insights](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-enable-app-insights#update-a-deployed-service).

![logs run](images/proyecto2_2/s3_logs_run.PNG)

"Application Insights enabled" is disabled before executing logs.py.

![enable insights](images/proyecto2_2/s3_application_insights_enabled.PNG)

### Consume an Azure Machine Learning model deployed as a web service

Another thing that is neccessary when working on a project formed by more colleagues is a intuitive way to interact with the deployed model for tests or just for compare with another model. For that kind of scenarios we can use swagger to automatically generate the expected inputs/outputs that the model handles.

#### Swagger documentation service

To interact with the swagger documentation tool, first download the swagger.json file stored in 'swagger uri' property located in your 'endpoints' section. Save the file in your local swagger folder. Execute the file 'swagger.sh' which will download the latest swagger container and run it on port 80 (if you don't have the permissions required, change port 80 to 9000). Execute the file 'serve.py' located in the swagger folder which will read the valid swagger.json file and start a local server to interact with the swagger documentation. Click [here](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-consume-web-service?tabs=python) for more information.

>*Note*: Once the swagger documentation is up in your local machine, you'll be able to interact with the documentations but won't be able to send any payloads to the deployed instance. This is due swagger is running on your local machine and using relative paths to access to the '/score' path. To switch this behavior you have to change the 'server' path in your swagger.json file.

![swagger interact](images/proyecto2_2/s5_swagger_interact.PNG)

#### Consuming an endpoint

To test your service endpoint open the 'endpoint.py' file and replace the **scoring_uri** and **key** variables with the values of the score uri and primary key located in the consume tab of your published endpoint model respectively to have permission to access to the service. This file will request to the endpoint service to predict 2 payload instances in the required json format and return their predicted labels. If you receive an error instead, make sure that the consume service values are the same in your script variables.

![run endpoint](images/proyecto2_2/s6_endppoint_run.PNG)

#### Endpoint service benchmark

Sometimes we also want to measure the time required in our system to respond to certain amount of queries. For this purpose azure CLI has a benchmark command 'ab' to run some payload *n* times and return basic statistics about the time it took to respond them.

Before executing the 'benchmark.sh' file it is necessary first to replace the **bearer** and **uri** values with the service uri link and primary key respectively. Executing this command will return some summary statistics of time from your service.

![apache benchmark](images/proyecto2_2/s6_apache_benchmark.PNG)

### Creating a pipeline automl 

[Pipelines](https://docs.microsoft.com/en-us/azure/machine-learning/concept-ml-pipelines) are useful parts for a successful CD/CI paradigm. They can be sticked together to form a more complex and parallel workflow in a team project. They can also help to re-train or re-run models whenever certain amount of time is given or some change to the scripts or databases is detected.

For this project we construct the pipeline instance using the python SDK contained in the included notebook. Execute all cells corresponding to 'aml-pipelines-with-automated-machine-learning-step.ipynb' notebook, only replacing the compute cluster and experiment names to save resources.

In cell:
```python
from azureml.pipeline.core import Pipeline

pipeline = Pipeline(
    description="pipeline_with_automlstep",
    workspace=ws,    
    steps=[automl_step])
```

You create a azure ML pipeline passing a previously 'AutoMLStep' object in which an automl already configured to be passed as a pipeline argument is constructed. Once the experiment is done a new pipeline will appear in your 'Pipelines' section.

#### Publishing a pipeline as a deployed service

You can deploy a finished pipeline as well. On the notebook this is achieved by the following cell.

```python
published_pipeline = pipeline_run.publish_pipeline(
    name="Bankmarketing Train", description="Training bankmarketing pipeline", version="1.0")
```

Where the arguments passed to the method are just the name, description and version of the pipeline endpoint.

![created pipeline](images/proyecto2/13_pipeline_created.PNG)

You can go to 'Pipeline endpoint' tab next to 'Pipeline endpoint' to see a list of all published pipelines so far.

![endpoint pipeline](images/proyecto2/14_pipeline_endpoint.PNG)

In your azure ML portal you can click on published pipelines to see their details. The following image depicts the diagram in azure ML of the dataset given to the autoML module along with the published pipeline overview showing such pipeline with status as 'active'. Ready to receive new requests.

This code cell shows how you can send your experiment to train and also see its progress within the notebook:

```python
from azureml.pipeline.core.run import PipelineRun
from azureml.widgets import RunDetails

published_pipeline_run = PipelineRun(ws.experiments["pipeline-rest-endpoint"], run_id)
RunDetails(published_pipeline_run).show()
```

where the arguments sent to PipelineRune are the experiment object and the run ID if the pipeline.

![run details](images/proyecto2/17_rundetails.PNG)

Once the pipeline experiment is completed you will end up with a pipeline endpoint which you can request for prediction queries like in normal automl projects. You can access to the pipeline overview to get important details like Status or REST endpoint uri. All this obtained through the azure SDK or web portal.

![bankmarketing automl](images/proyecto2/15_bankmarketing_automl.PNG)

on the last part of this project you end up with a scheduled pipeline endpoint with a model trained on the bankmarketing dataset, ready to classify new observations using a request JSON payload.

![scheduled run](images/proyecto2/18_scheduled_run.PNG)

Entire pipelines can be deployed in matter of minutes wraping a lot of overhead for the machine learning/data science teams, leaving more time for experimentation.

## Screen Recording
<br>

[Project screencast](https://youtu.be/IrqmbhqXVtE)

## Standout Suggestions
<br>

In this project we absolutely relied on autoML to feature engineering but maybe a better approach would have been to tackle this part ourselves. Suggestions for future improvements would be approach this part, usind EDA to maybe gain a better undersanding about the nature of the problem. Another possible approach to increase de performance is to include deep learning models. Deep learning methods are known to make implicit feature engineering, applying different non-linear transformation functions.

---
title: "Back to the future: MLflow"
datePublished: 2023-11-19T03:11:02.576Z
cuid: clp4wi4kw000108jtd65c9sl3
slug: back-to-the-future-mlflow
tags: data-science, mlops, mlflow, azure-machine-learning

---

### **What is MLflow?**

MLflow is an open-source platform for managing the machine learning lifecycle. It provides a set of tools and APIs tracking experiments, deploying models, and managing model versions. MLflow is provider agnostic and can be used with Azure, AWS or GCP.

### Why MLflow?

When I joined the Data Science "team", we had a (*broken*) data pipeline to get training data (*more on this some other day)* and a model deployed as "Python script" (yes, as a script, which gets executed as command line, loading pickled model for every scoring request) in Azure Container Instance.

Besides Github repository with training and scoring code, and pickled model in shared drive, there was no information available on how the model was trained or what is required to retrain a model. To make matter worse, it took someone to spend days in recreating the environment without any success and when it came to redeploy the container to update API contract, I had to manually create a new docker image using the one running and manually overwrite DLL files.

It became quite clear to me that without proper MLOps in place, things will get even worse. This is where MLflow came to into play, it offered:

* **Experiment tracking:** MLflow provided comprehensive experiment tracking capabilities, offering Azure Machine Learning workspaces as a centralized repository (albeit with some additional work) for experiment details, parameters, and metrics.
    
* **Model Packaging and Versioning:** Managing models in a consistent format is important for sharing and deploying across different environments. MLflow simplified this process, allowed us to package our models in efficient, consistent and reproducible manner.
    
* **Collaborative Development:** With MLflow, collaborative development became seamless, easing the burden of environment recreation. By leveraging MLflow, we were able to easily recreated the exact environment used for training, saving significant time and effort.
    
* **Efficient Model Deployment:** MLflow with it's integration with Azure Machine Learning workspace made deployment to Azure Container Instance simpler and consistent. No more manual Docker image creation, copying pickle files around or overwriting DLL files.
    
* **Facilitated Retraining:** MLflow's support for packaging code and dependencies made it easier to reproduce machine learning environments.
    

### **Why did we move away from MLflow?**

Since everything from our operational sources to reporting was already on Azure, it was only natural for us to choose Azure Machine Learning workspaces to manage our machine learning experiments.

This is where Azure ML SDK v1 proved easier to setup and use then MLflow:

* Azure ML SDK v1 provided everything we needed from experiment logging to model deployment and scoring.
    
* Running experiments locally with MLflow tracking them to Azure Machine Learning workspace turned out of be quite problematic and required fair bit of bootstrap code.
    
* SDK gave us multiple options for deploying models without much effort. We could deploy models to Azure Container Instances, Azure Kubernetes Service, or even locally.
    

### **Why are we back to MLflow?**

MLflow is the only and recommended way to log metrics, parameters and files in Azure ML SDK v2. As Azure Machine Learning workspaces are MLflow-compatible, now it is as easy as code below to start MLflow tracking:

```python
mlflow.set_experiment("model-v1")
mlflow_run = mlflow.start_run()
```

Besides easy of use and no dependency on Azure Machine Learning in training routines:

* Using MLflow makes training routines cloud-agnostic. This means that we can train our models on any cloud platform or even locally.
    
* Provides consistent set of tools regardless of where experiments are running.
    
* We can use Azure Machine Learning workspaces as our tracking server, even if it runs on AWS or, as a matter of fact, anywhere - as long as MLflow is configured to point to the workspace where the tracking should happen.
    

In addition to these benefits, MLflow is opensource, mature and feature-rich platform for end-to-end MLOps with over 13 million monthly downloads. It has a large and active community, and it is constantly being updated with new features.
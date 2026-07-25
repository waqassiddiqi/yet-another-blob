---
title: "Journey Through the MLOps Maturity Model"
datePublished: 2025-08-28T14:43:22.272Z
cuid: cmevikgtc000302jpazatcx4j
slug: journey-through-the-mlops-maturity-model
tags: ai, azure, data-science, machine-learning, mlops, mlflow, azureml, dataops, azure-machine-learning

---

When I first joined Cash Converters, the data science setup was pretty raw. There was no proper repository for training data or models. Just a few Python files and few pickled model. That was it.

Reproducibility was non-existent. There was no way to figure out which version of data had been used to train a model. If someone asked anyone from a Data Science & AI teams to rerun an experiment, they couldn’t. Traceability was also missing—we had a model in production, good, how it went there, no clear answer.

And deployment? It was as fragile as it gets. A Python script running inside an ASP.NET scoring service container, loading the pickle file on every inference request. Every prediction started with reloading the model. Slow, brittle, and error-prone - restarting Azure Container Instance was not running on periodically was normal.

When I first started working on automating this entire process, I didn’t know about the **MLOps Maturity Model from Microsoft** ([link](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/mlops-maturity-model?utm_source=waqassiddiqi.com)). What I did know—coming from a software engineering background—was that we couldn’t keep going with manual scripts, no reproducibility, and fragile deployments.

For me, the natural step was to bring in CI/CD practices being used for years in software development. Automate whatever could be automated. Track everything. Make deployments predictable.

Quite late in our journey of automating ML, we came across maturity model, and it perfectly described the path we had already taken. We had unknowingly moved through the levels—starting at **Level 0 (No MLOps)** and gradually building toward higher levels of maturity.

## Level 0 → Level 1

The first step was to at least bring in some DevOps practices. Since our code was already in GitHub, we started with **GitHub Actions**. That gave us automated builds and tests for the service, but the ML side was still manual. This was basically **Level 1 (DevOps but no MLOps)**.

## Level 2: Automated Training

Things started to change when we brought in **Azure ML SDK** and **MLflow**.

* With Azure ML pipelines, training became reproducible.
    
* With MLflow, every run was logged—parameters, metrics, artifacts. Suddenly, Data Scientist could compare experiments and track what was happening.
    
* Data and models were versioned properly, finally giving us traceability.
    

This was the first big win. Moving from “I have no idea what’s running” to “I know exactly which model was trained, when, and with which data.”

## Level 3: Automated Deployment

Next came deployment. No more copying pickle files into a container.

Now, when a pipeline finishes, the trained model is registered in Azure ML. From there, **GitHub Actions** takes over—deploying the model to staging or production environments. Approvals are built in, so we can control when things go live.

This was a game-changer. Deployments went from fragile manual steps to a reliable process that just works.

## Level 4: Full MLOps

We’re not fully there yet, but we’re moving in the right direction. Monitoring and retraining are being wired up. Azure ML SDK provides functionality to implement data drift & data quality monitoring and prediction drift in production.

---

## Why We Chose Azure ML + MLflow (and Why We didn’t stick with Databricks Registry)

When we were evaluating tools, we initially tried **MLflow + Databricks Model Registry**. It gave us basic versioning, but it was tightly coupled with the Databricks ecosystem and always had issues setting up tracking URI and logging experiments running locally.

**Azure ML** offered a more **centralized model registry** that fit naturally with our wider Azure ecosystem (we were already using Azure heavily in other part of the business). It also provided out-of-the-box support for deployment endpoints and (evolving) but excellent Python SDK.

**MLflow stayed in the picture** for several reasons:

* It was becoming the *de facto* standard in the community.
    
* Extensive documentation.
    
* Excellent support of saving / loading custom models.
    
* Lightweight and easy for data team to adopt.
    

As of today, Azure ML has become our **production-grade platform**, while MLflow remained our **experiment tracking layer**. This combination gave us the best of both worlds.

---

## What I Learned Along the Way

Looking back, a few things stand out:

* **Start small.** Even automating one pipeline reduces headaches.
    
* **Traceability matters.** The moment we had lineage, conversations changed. We could stop guessing and start deciding.
    
* **CI/CD isn’t just for app code.** Treating ML like software made everything more stable.
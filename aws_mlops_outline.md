# AWS MLOps Overview

> MLOps is the end-to-end process of *building* and *deploying* models. \([source](https://pynomial.com/2021/09/what-are-the-steps-in-mlops/)\)

## Stages of MLOps

While there's no single set of components to an MLOps pipeline as each workflow has different parameters and objectives, the list below comprises a generally agreed-upon breakdown of the primary aspects of both building and deploying a model in an MLOps pipeline.

### Build

1. Data Prep
   1. Environment & Pipeline Configuration
   2. Data Collection, Selection, and Verification
   3. Data Versioning
   4. Algorithm & ML Framework Selection
   5. Feature Engineering
   6. Experimentation (Tuning) & Model Versioning
2. Trained Model
3. ML Code Testing

### Deploy

1. Rollout
2. Request Logging & Audit Trail
3. Process Management
   1. Resource Management
   2. Data / Metadata Management
4. Monitoring
   1. Accuracy
   2. Drift
   3. Outliers
   4. Explanations

<BR>

## MLOps Solutions: Build vs. Buy? 

Use case, resources, and department structure will determine whether purchasing a pre-packaged MLOps solution is worth the cost for an organization.  Unsurprisingly, the market is competitive for "one-stop shop" MLOps solutions.  The fundamental appeal of these offerings is that they make MLOps "easy" and have solved many of the headaches for you already by bundling various components together.  Think "tech hand-holding."  

But, as is true for all proprietary platforms, hitching your wagon to one of these offerings might mean limiting your team's options in both usable tech stack and in problem solving flexibility.  If your team would like to alter something substantial in your MLOps workflow in the future, your pre-packaged platform might prohibit you from doing so as it isn't built to work with a given product or platform that you feel would greatly improve your workflow.  

There are two key realizations to make regarding a pre-baked solution:

1. If you have a valid data engineering team, you are likely capable of building an MLOps pipeline that is tailored for your needs.  The engineers and scientists will also learn how to manage this pipeline more effectively by building it.
2. 
3. Even with a commercial solution, you will still need to do some engineering and configuration that requires a level of proficiency (e.g. my mother could not license a commercial platform and set it up properly).

These two points suggest to me that most data teams are likely better served by using open-source tools and building their own MLOps pipeline.  \([source](https://towardsdatascience.com/mlops-the-ultimate-guide-9d902c752fd1)\)

ToDO: List some popular options

---

## Data Science on AWS

MLOps requires the collaboration of multiple teams, usually centered on some mixture of data science to deliver the trained model, DevOps engineers to manage the infrastructure that hosts the model as a REST API, and application developers to integrate the REST API into the applications.  What follows below are excerpts from \([this book](https://smile.amazon.com/Data-Science-AWS-End-End/dp/1492079391/ref=sr_1_1?keywords=data+science+on+aws+o%27reilly&qid=1664413319&qu=eyJxc2MiOiIwLjIxIiwicXNhIjoiMC4wMCIsInFzcCI6IjAuMDAifQ%3D%3D&sprefix=data+science+on+aws%2Caps%2C140&sr=8-1)\) about Data Science and MLOps on AWS.

## Best Practices

* Data-quality checks
  * Implement repeatable data-quality tests
* Define model performance metrics
  * Map metrics to business objectives and continuously monitor these KPIs. Set thresholds which denote invalid performance and trigger model retraining.
* Track and version everything
  * Track model development through experiments and lineage. Version datasets, feature engineering code, hyperparameter tunings, and trained models.
* Choose task-specific resources
  * Model training might have different resource requirements than model serving.
* Continuous monitoring
  * Model drift and data drift should be overseen and models retrained as needed
* Automate workflows
  * More automation means less human error and more time for humans to focus on solving problems.  

## Data Security

Data security, or even SecOps more broadly, is a critical requirement for any MLOps pipeline. It is commonly considered good practice to have some degree of restricted access to data-labeling jobs, data-processing scripts, models, end-points, and batch jobs.  Data governance should include implementing a data lineage, which tracks data transformations applied to training data.  This is all assuming approved data compliance practices.

## Reliability

Using the data lineage practices noted above makes recovery from service disruptions or failures much easier and faster by allowing the re-creation of exact versions of data and a model. We want to build once and use the model artifacts to deploy the model across multiple AWS accounts and environments.

## Performance Efficiency

Depending on the use case, different amounts or types of compute (including GPU or 'deploying at the edge', i.e. on-device) might increase efficiency.  Every company wants to save costs, so understanding how much efficiency is gained by utilizing different compute is imperative as spending a bit more up front on compute can lead to less overall usage time and thus actually reduce total cost.  Regulations for specific types of data might also require deploying on the edge instead of keeping sensitive information in the cloud.  It's important to understand how these factors that impact efficiency can also impact cost.

## AWS Cost Savings

AWS offers various "Savings Plans" which seek to optimize compute for the tasks and timing windows needed.

## Software Pipelines

*Jenkins* is currently the most popular open source tool for managing CI/CD pipelines.  It allows for reports on the pipeline at any point along the pipeline's execution ans well as providing mechanisms to restart any failed components.  

---

## Machine Learning Pipelines

As much as possible, automating the various steps in an ML pipeline reduces error, alerts for drift, and allows easier modifications downstream.  Running one-off Python scripts is likely to cause human error, slowed (and inflexible) development, and inconsistent results downstream. AWS has done a good job of building tools into SageMaker that directly manage ML pipelines.  The table below gives an example of the built-in tools available in SageMaker.

ML Pipeline                     | AWS Tool
--------------------------------|------------------------------------
Data ingestion & versioning     | S3 & Data Wrangler
Data analysis & validation      | SageMaker Studio Notebooks
Feature selection & engineering | SageMaker Processing Jobs
Model training & tuning         | SageMaker Training & Tuning
Model analysis & versioning     | S3 & SageMaker Debugger
Model deployment & monitoring   | SageMaker Endpoints & Batch Predict

### AWS Native Tool: SageMaker Pipelines

SageMaker Pipelines [home page](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-sdk.html).

SageMaker has a Python SDK that allows the creation of programmatic pipelines.
This SDK contains tools for the entire ML pipeline, including data ingestion & processing, feature engineering, model training & evaluation, and model deployment.

A sampling of packages or functions used in a SageMaker Pipeline workflow:
`ProcessingStep`, `TrainingStep`, `PropertyFile`, `MetricsSource`, `RegisterModel`, `CreateModelInput`, `CreateModelStep`, `ConditionStep`, `Pipeline`, etc.

The point of this list is to show that the SageMaker Pipeline offers a fully featured approach to managing ML Pipelines in AWS.  Each step along the way is reviewable in an Artifact Lineage table, showing things like the executed script, whether it was an input or output for the pipeline, and what type of object it was, such as dataset, image, model, etc.

A pipeline can be automated to start using either...

1. event-based triggers (such as a new data file hitting S3)
2. time-based triggers
3. threshold-based triggers (such as prediction accuracy % or drift)

For event-based triggers in S3, AWS offers *CouldTrail* data-logging.  The next step is the create an Amazon *EventBridge* rule that triggers the SageMaker Pipeline when new files hit an S3 bucket.  For the time being, to actually check if this pipeline-triggering rule is matched, an AWS *Lambda* function is used.  Once it is matched, the pipeline begins.  However, in the near future it is likely that AWS will natively be able to evaluate pipeline-triggering rules via EventBridge, meaning that writing AWS Lambda functions will not be required.

This general concept holds for time-based and threshold-based triggers. Threshold-based triggers use AWS *Clarify* to continuously monitor deployed models in set metrics.

### Other ML Pipeline Tools

Apart from SageMaker Pipelines, there is another AWS pipeline orchestrator as well as open source tools which are designed to integrate with AWS.  

#### AWS Step Functions

*Step Functions* were not designed specifically for ML pipelines, but can orchestrate many of the same workflows as SageMaker Pipelines and also exposes the user to the Step Functions Data Science SDK.  (Note: I am not personally familiar with this SDK, so I am not sure what functionality it offers that might be desirable compared to SageMaker). I'm unsure where Step Functions differ from SageMaker Pipelines from implementation except that they are more broad in scope -- that is, they aren't built to operate only with SageMaker.

#### Kubeflow

An open source tool, Kubeflow is a companion tool to Kubernetes, making it a potentially appealing option for teams already using Kubernetes to manage their distributed workflows.  Functionally, Kubeflow allows comparable orchestration and monitoring to either of the AWS-native options above.  However, it also means that extra time is required to manage Kubernetes itself, an additional time sink and technical drain for pipelines that could be run in AWS tools alone.  Thus, the appeal of Kubeflow is primarily dependent upon a team's existing usage of Kubernetes.

#### Apache Airflow

Initially developed for engineering and ETL analytics pipelines, Airflow has matured into an ML pipeline orchestrator. Amazon supports *Amazon Managed Workflows* (MWAA) for Apache Airflow to simplify its use on AWS. The maturity of the Airflow platform means it has a large library of plug-ins and an established native integration with many AWS services.  Like Kubeflow and Kubernetes above, using Airflow to manage an AWS ML pipeline could make good sense if Airflow is already being used to manage the ETL or engineering pipelines. Extending one platform to cover more of the total project workflow can reduce technical overhead and allow for teams to stretch their expertise further due to already having familiarity with the given platform.

#### MLflow

A more specialized open source platform that focuses on experiment tracking and multi-framework support, including Spark.  It is limited as a full workflow manager.  It also requires a team to build and maintain their own EC2 and EKS clusters.  It currently excels mostly as a simple experiment tracker.

#### TensorFlow Extended (TFX)

TFX is actually just an open source collection of Python libraries used within popular orchestrators like the ones listed above.  Designed to be used with TensorFlow ML, TFX can be helpful in adding structure to these pipelines.  Technically it offers loose support for other platforms, such as sckit-learn, but might not be as reliable for fully-featured for them.  Scaling TFX beyond a single node requires use of *Apache Beam*, yet another distributed data processing platform known to have a bit of a learning curve.  For these reasons, TFX tends to find usage predominantly in TensorFlow projects.

#### Human-in-the-loop

Certain types of ML workflows either require a human-in-the-loop step or find a benefit from one. AWS offers two services to aid in this endeavor: *A2I* and *SageMaker Ground Truth*. A2I allows ML teams to integrate human review workflows into their applications.  SageMaker Ground Truth allows humans to create accurate training datasets.  Their technical implementations vary and Ground Truth has a UI (think: image tagging or data labeling).

### Step Caching and Cost Reduction

SageMaker Pipelines can cache steps that prevent reexecuting steps that have not changed since the previous run, sparing wasted compute and time.  Functionally, SageMaker Pipelines work by building upon primitives such as SageMaker Training Jobs.  *Spot Instances* allow for comparing time and cost savings across various pipeline steps and runs. 
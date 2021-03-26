## Pipelines and components
[Source](https://www.kubeflow.org/docs/pipelines/overview/pipelines-overview/)
### Component:
- A component performs a specific task, it is analogous to a function in that it has a name, parameters, return values, and a body.
- A pipeline component is a self-contained set of user code, packaged as a Docker image, that performs one step in the pipeline.
- Components are composed of a set of input parameters, a set of outputs, and the location of a container image. A component's container image is a package that includes the component's executable code and a definition of the environment that the code runs in.
- A component has 2 parts:
	- Client code: The code that talks to endpoints to submit jobs. For example, code to talk to the Google Dataproc API to submit a Spark job.

	- Runtime code: The code that does the actual job and usually runs in the cluster. For example, Spark code that transforms raw data into preprocessed data.

	- Note the naming convention for client code and runtime code—for a task named “mytask”:
		* The mytask.py program contains the client code.
		* The mytask directory contains all the runtime code

### Pipeline
- A pipeline is a description of an ML workflow, including all of the components in the workflow and how they combine in the form of a graph. 
- The pipeline includes the definition of the inputs (parameters) required to run the pipeline and the inputs and outputs of each component.


## Kubeflow Pipelines with AI Platform
[Qwiklabs course](https://www.qwiklabs.com/focuses/10948?parent=catalog)
[Tutorial Notebook from Qwiklabs](https://github.com/kubeflow/pipelines/blob/master/samples/core/ai_platform/ai_platform.ipynb)
### Setting-up
- KFP are deployed on GKE clusters. GKE clusters run on VMs. Hence, once you create a GKE cluster, it will run on multiple (2 by default) VMs.
	- You can check cluster on GCP console > kubernetes engine > clusters 
	- You can check the VMs running the cluster on GCP console > VMs > VM instances
- A KFP is an application on GKE
	- You can check that on Console > kubernetes > applications 
- You can connect this  pipeline instance from Python client via Kubeflow Pipeline SDK using:
	```
	import kfp
	client = kfp.Client(host='6b776daa35b2c90e-dot-us-central2.pipelines.googleusercontent.com')
	```
- We use GCS to store output of pipelines. Hence, we also need to create GCS bucket before using pipelines. 
- To create GCS bbucket
	```
	gsutil mb -l ${REGION} gs://${PROJECT}
	```	
	`mb` means make bucket. Here we are creating bucket with the same name as project name. 	
	
### Developing Pipeline
- We use KFP DSL (Domain Specific Language) to build pipeline up.
- We can build pipeline in AI Platform Notebook and can use various pre-built components to interact with other GCP products such as BigQ, AI Platform Training etc. 
	- This will run all the computation on the respective services rather than running them on GKE cluster locally.
- Building a pipeline:
	```
	@dsl.pipeline(
    name=PIPELINE_NAME,
    description=PIPELINE_DESCRIPTION
	)

	def pipeline(args):
		# use various components.
		return True
	```
- Any built pipeline is than submitted to KFP instance to run. For this, we use
	-
	```
	pipeline = kfp.Client(host='<HOST NAME>').create_run_from_pipeline_func(pipeline, arguments={})
	```
	**OR**
	Once pipeline is developed as python file, you can use CLI to compile:
	```
	!dsl-compile --py pipeline_python_file.py --output pipeline_python_file.yaml
	```
	Compiling a pipeline means transforming your pipelone's Python code into a static configuration (YAML).
	
	### Deploying pipeline package
	* `ENDPOINT` set the ENDPOINT constant to the endpoint to your AI Platform Pipelines instance. Then endpoint to the [AI Platform Pipelines](https://console.cloud.google.com/ai-platform/pipelines/clusters) instance can be found on the AI Platform Pipelines page in the Google Cloud Console.
	* `PIPELINE_NAME`: Any name you want to give to the pipeline
	```
	!kfp --endpoint $ENDPOINT pipeline upload \
	-p $PIPELINE_NAME \
	pipeline_python_file.yaml
	```
	This will return pipeline ID.
	
	### Running pipeline
	```
	!kfp --endpoint $ENDPOINT run submit \
	-e $EXPERIMENT_NAME \
	-r $RUN_ID \
	-p $PIPELINE_ID \
	## And other arguments required for your pipeline##
	```
**TO UNDERSTAND THE PROCESS WITH EXAMPLE GO [HERE](https://github.com/kaushil24/mlops-on-gcp/blob/master/workshops/kfp-caip-sklearn/lab-02-kfp-pipeline/lab-02.ipynb)**

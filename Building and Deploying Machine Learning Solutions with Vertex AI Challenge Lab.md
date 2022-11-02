## Building and Deploying Machine Learning Solutions with Vertex AI: Challenge Lab:-

### Set up env parameters and download the necessary files :-

```yaml
export PROJECT=$(gcloud info --format='value(config.project)')

```

### Task - 1 : CCreate an Vertex Notebooks instance :-

Vertex AI > Workbench > User-Managed Notebooks  
Create a Notebook instance  
Name your notebook vertex-ai-challenge

### Task - 2 : Download the challenge notebookn :-
In your notebook, click the terminal.
```yaml
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
cd training-data-analyst/quests/vertex-ai/vertex-challenge-lab
pip install -U -r requirements.txt

```
Go to the enclosing folder: training-data-analyst/quests/vertex-ai/vertex-challenge-lab.  

Open the notebook file vertex-challenge-lab.ipynb.  

In the Setup section, define your PROJECT_ID, GCS_BUCKET, and USER variables  

### Task - 3 : Build and train your model locally in a Vertex notebook :-

TODO: Create a globally unique Google Cloud Storage bucket for artifact storage.GCS_BUCKET = 
```yaml
f"gs://<PROJECT_ID>-vertex-challenge-lab"
```

TODO: Add a hub.KerasLayer for BERT text preprocessing using the hparams dict.   
Name the layer 'preprocessing' and store in the variable preprocessor

```yaml
preprocessor = hub.KerasLayer(hparams['tfhub-bert-preprocessor'],name='preprocessing')
```
TODO: Add a trainable hub.KerasLayer for BERT text encoding using the hparams dict.  
Name the layer 'BERT_encoder' and store in the variable encode
```yaml
encoder = hub.KerasLayer(hparams['tfhub-bert-encoder'], trainable=True,name='BERT_encoder')
```

TODO: Save your BERT sentiment classifier locally
```yaml
"model-dir": "./bert-sentiment-classifier-local"
```
### Task - 4 : Use Cloud Build to build and submit your model container to Google Cloud Artifact Registry :-

TODO: create a Docker Artifact Registry using the gcloud CLI. Note the requiredrespository-format and location flags
```yaml
!gcloud artifacts repositories create {ARTIFACT_REGISTRY} \
--repository-format=docker \
--location={REGION} \
--description="Artifact registry for ML custom training images for sentiment classification"
```

TODO: use Cloud Build to build and submit your custom model container to your ArtifactRegistry 
```yaml
!gcloud builds submit {MODEL_DIR} --timeout=20m --config {MODEL_DIR}/cloudbuild.yaml
```

### Task - 5 : Define a pipeline using the KFP SDK :-

TODO: fill in the remaining arguments from the pipeline constructor.
```yaml
USER = "qwiklabsdemo
display_name=display_name,container_uri=container_uri,model_serving_container_image_uri=model_serving_container_image_uri,base_output_dir=GCS_BASE_OUTPUT_DIR,
```
TODO: Generate online predictions using your Vertex Endpoint

### Task - 6 : Deploy a Cloud Dataflow Pipeline :-
```yaml
endpoint = vertexai.Endpoint(
    endpoint_name=ENDPOINT_NAME,
    project=PROJECT_ID,
    location=REGION
    )    
```

TODO: write a movie review to test your model e.g. "The Dark Knight is the best Batmanmovie!
```yaml
test_review = "The Dark Knight is the best Batman movie!" 

```
TODO: use your Endpoint to return prediction for your test_review.
```yaml
prediction = endpoint.predict([test_review])

```

Congratulations! :beer: :beer:
## Automate Interactions with Contact Center AI: Challenge Lab :-

### Set up env parameters and download the necessary files :-

```yaml
export PROJECT=$(gcloud info --format='value(config.project)')
export BUCKET=e65bf6f9-2069-462d-97c3-2d5eb9999999
export REGION=us-central1
export DATASET=saf_877
export TOPIC=saf-topic-112
export DATAFLOW=DFaudio-585 
export TMP_BUCKET=831a9adc-2046-47b3-a401-9071deeeeee

git clone https://github.com/GoogleCloudPlatform/dataflow-contact-center-speech-analysis.git
```

### Task - 1 : Create a Regional Cloud Storage bucket :-

```yaml
gsutil mb -l ${REGION} gs://${PROJECT}/
```

### Task - 2 : Create a Cloud Function :-

Go to the “saf-longrun-job-func” directory.

```yaml
gcloud functions deploy safLongRunJobFunc --runtime nodejs12 --trigger-resource ${PROJECT} --region ${REGION}  --trigger-event google.storage.object.finalize
```

### Task - 3 : Create a BigQuery Dataset :-

```yaml
bq mk ${DATASET}
```

### Task - 4 : Create Cloud Pub/Sub Topic :-

```yaml
gcloud pubsub topics create ${TOPIC}
```

### Task - 5 : Create a Cloud Storage Bucket for Staging Contents :-

```yaml
gsutil mb -l ${PROJECT} gs://${PROJECT}/
mkdir ${DATAFLOW}
```

### Task - 6 : Deploy a Cloud Dataflow Pipeline :-

Go to the saf-longrun-job-dataflow directory.

```yaml
python -m virtualenv env -p python3
source env/bin/activate
pip install apache-beam[gcp]
pip install dateparser
python saflongrunjobdataflow.py \
--project=[PROJECT_ID] \
--region=${REGION} \
--input_topic=projects/${PROJECT}/topics/helpdesk \
--runner=DataflowRunner \
--temp_location=gs://${TMP_BUCKET}/${DATAFLOW} \
--output_bigquery=${PROJECT}:saf.transcripts \
--requirements_file=requirements.txt
```

### Task - 7 : Upload Sample Audio Files for Processing :-

Mono flac audio sample :-

```yaml
# mono flac audio sample
gsutil -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:false -h x-goog-meta-pubsubtopicname:${TOPIC} -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_mono.flac gs://${BUCKET}
```

Stereo wav audio sample :-

```yaml
# stereo wav audio sample
gsutil -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:true -h x-goog-meta-pubsubtopicname:${TOPIC} -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_stereo.wav gs://${BUCKET}
```

After uploading sample audio files for processing, run this query in BigQury to check (change [DATASET] to given name) :-
```yaml
select * from [DATASET].transcripts
```

### Task - 8 : Run a Data Loss Prevention Job :-

Run the following query in the BigQury query editor :-

```yaml
select * from (SELECT entities.name,entities.type, COUNT(entities.name) AS count 
FROM [DATASET].transcripts, UNNEST(entities) entities 
GROUP BY entities.name, entities.type ORDER BY count ASC ) Where count > 5
```

Click Save table to Bigquery, give a name.   
Go to the new table, click EXPORT > SCAN WITH DLP and start job.  
Wait til the job completed (3-5min)

Congratulations! :beer: :beer:
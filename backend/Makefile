# To run this make file you will need to give the next arguments. PROJECT_ID, DATASET_NAME, TOPIC_NAME and SUBSCRIPTION_NAME.
# If you are lazy (like me), you can just set them up in the following lines :-)

# PROJECT_ID=<YOUR_PROJECT_ID>
# DATASET_NAME=<YOUR_DATASET_NAME>
# TOPIC_NAME=<YOUR_TOPIC_NAME>
# SUBSCRIPTION_NAME=<YOUR_SUBSCRIPTION_NAME>
# SCHEMAS_BUCKET=<YOUR_BANIAS_SCHEMAS_BUCKET>
# TEMP_BUCKET=<YOUR_TEMP_BUCKET>
# TEMPLATE_BUCKET=<YOUR_TEMPLATE_BUCKET>
# TEMPLATE_FILE=<YOUR_TEMPLATE_FILE_NAME>
# DATASET_LOCATION=<YOUR_DATASET_LOCATION>

TABLE_NAME=banias
TEMP_FOLDER=tmp
STAGING_FOLDER=pipeline-staging
EVENTS_SUBSCRIPTION_PATH="projects/$(PROJECT_ID)/subscriptions/$(SUBSCRIPTION_NAME)"
ERRORS_TABLE_NAME=error
DATASET=banias


env_setup: _create_buckets _create_topic _create_subscription _create_dataset

install:
	mvn install

build: install
	mvn package

run_local: build
	mvn exec:java -Dexec.mainClass=com.doitintl.banias.BaniasPipeline \
	-Dexec.cleanupDaemonThreads=false \
	-Dexec.args=" \
	--project=$(PROJECT_ID) \
	--tempLocation=gs://$(TEMP_BUCKET)/ \
	--gcpTempLocation=gs://$(TEMP_BUCKET)/$(TEMP_FOLDER) \
	--runner=DirectRunner \
	--defaultWorkerLogLevel=DEBUG \
	--eventsSubscriptionPath=$(EVENTS_SUBSCRIPTION_PATH) \
    --errorsTableName=$(ERRORS_TABLE_NAME) \
    --GCSSchemasBucketName=$(SCHEMAS_BUCKET) \
    --dataset=$(DATASET) \
    --numWorkers=3 \
	"

run: build
	mvn exec:java -Dexec.mainClass=com.doitintl.banias.BaniasPipeline \
	-Dexec.cleanupDaemonThreads=false \
	-Dexec.args=" \
	--project=$(PROJECT_ID) \
	--tempLocation=gs://$(TEMP_BUCKET)/ \
	--gcpTempLocation=gs://$(TEMP_BUCKET)/$(TEMP_FOLDER) \
	--stagingLocation=gs://$(TEMP_BUCKET)/$(STAGING_FOLDER) \
	--runner=DataflowRunner \
	--defaultWorkerLogLevel=DEBUG \
	--eventsSubscriptionPath=$(EVENTS_SUBSCRIPTION_PATH) \
    --errorsTableName=$(ERRORS_TABLE_NAME) \
    --GCSSchemasBucketName=$(SCHEMAS_BUCKET) \
    --dataset=$(DATASET) \
    --numWorkers=3 \
	"

create_template: build
	mvn exec:java -Dexec.mainClass=com.doitintl.banias.BaniasPipeline \
	-Dexec.cleanupDaemonThreads=false \
	-Dexec.args=" \
	--jobName="BaniasPipeline" \
	--project=$(PROJECT_ID) \
	--tempLocation=gs://$(TEMP_BUCKET)/ \
	--gcpTempLocation=gs://$(TEMP_BUCKET)/$(TEMP_FOLDER) \
	--stagingLocation=gs://$(TEMP_BUCKET)/$(STAGING_FOLDER) \
	--templateLocation=gs://$(TEMPLATE_BUCKET)/$(TEMPLATE_FILE) \
	--runner=DataflowRunner \
	--defaultWorkerLogLevel=DEBUG \
	--eventsSubscriptionPath=$(EVENTS_SUBSCRIPTION_PATH) \
    --errorsTableName=$(ERRORS_TABLE_NAME) \
    --GCSSchemasBucketName=$(SCHEMAS_BUCKET) \
    --dataset=$(DATASET) \
    --numWorkers=3 \
	"

### Helpers:
_create_buckets:
	gsutil ls -b gs://$(TEMP_BUCKET) || gsutil mb gs://$(TEMP_BUCKET)
	gsutil ls -b gs://$(SCHEMAS_BUCKET) || gsutil mb gs://$(SCHEMAS_BUCKET)
ifdef TEMPLATE_BUCKET
	gsutil ls -b gs://$(TEMPLATE_BUCKET) || gsutil mb gs://$(TEMPLATE_BUCKET)
endif

_create_dataset:
	bq ls | grep -w $(DATASET) || bq --location=$(DATASET_LOCATION) mk --dataset $(PROJECT_ID):$(DATASET)

_create_topic:
	gcloud pubsub --project $(PROJECT_ID) topics list | grep -w $(TOPIC_NAME) || gcloud pubsub --project $(PROJECT_ID) topics create $(TOPIC_NAME)

_create_subscription:
	gcloud pubsub --project $(PROJECT_ID) subscriptions list | grep -w $(SUBSCRIPTION_NAME) || gcloud pubsub --project $(PROJECT_ID) subscriptions create $(SUBSCRIPTION_NAME) --topic=$(TOPIC_NAME)

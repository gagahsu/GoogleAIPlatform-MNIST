# GoogleAIPlatform-MNIST
Describe how to use MNIST project on Gitlab 
# Clone the lab repository in the cloud shell, then cd into that dir.
git clone https://github.com/GoogleCloudPlatform/cloudml-dist-mnist-example
cd cloudml-dist-mnist-example
# Create a bucket used for training jobs.
PROJECT_ID=$(gcloud config list project --format "value(core.project)")
BUCKET="${PROJECT_ID}-ml"
gsutil mb -c regional -l us-central1 gs://${BUCKET}
# Insatll TensorFlow
sudo pip install tensorflow==1.2.1
# Submit a training job to Cloud AI Platform.
JOB_NAME="job_$(date +%Y%m%d_%H%M%S)"
gcloud ai-platform jobs submit training ${JOB_NAME} \
    --package-path trainer \
    --module-name trainer.task \
    --staging-bucket gs://${BUCKET} \
    --job-dir gs://${BUCKET}/${JOB_NAME} \
    --runtime-version 1.2 \
    --region us-central1 \
    --config config/config.yaml \
    -- \
    --data_dir gs://${BUCKET}/data \
    --output_dir gs://${BUCKET}/${JOB_NAME} \
    --train_steps 10000
--train_steps 選項指定訓練批次總數。
--config/config.yaml can resign machine source level
--setup.py 設定指令碼以在節點上安裝其他模組
--model.py 定義卷積類神經網路模型的 TensorFlow 程式碼。
--task.py 執行訓練工作的 TensorFlow 程式碼。在此範例中，使用 Experiment API 以分散方式執行訓練迴圈
# At the end of the training, the final evaluation against the testset is shown as below. In this example, it achieved 99.16% accuracy for the testset.
Saving dict for global step 10008: accuracy = 0.9916, global_step = 10008, loss = 0.0407254
# Model will be stored in the bucket
SavedModel written to: gs://mnist-258908-ml/job_20191115_094619/export/Servo/1573783257/saved_model.pb
# Deploy the trained model and set the default version.
MODEL_NAME=MNIST
gcloud ai-platform models create --regions us-central1 ${MODEL_NAME}
VERSION_NAME=v1
ORIGIN=$(gsutil ls gs://${BUCKET}/${JOB_NAME}/export/Servo | tail -1)
gcloud ai-platform versions create \
    --origin ${ORIGIN} \
    --model ${MODEL_NAME} \
    ${VERSION_NAME}
gcloud ai-platform versions set-default --model ${MODEL_NAME} ${VERSION_NAME}
# Create a JSON request file.
./scripts/make_request.py
# Submit an online prediction request.
gcloud ai-platform predict --model ${MODEL_NAME} --json-instances request.json
# Launch Datalab from the Cloud Shell.
datalab create mnist-datalab --zone us-central1-a
# Open a new notebook and execute the following command.
%%bash
wget https://raw.githubusercontent.com/GoogleCloudPlatform/cloudml-dist-mnist-example/master/notebooks/Online%20prediction%20example.ipynb

# Reference :
https://cloud.google.com/ml-engine/docs/tensorflow/distributed-tensorflow-mnist-cloud-datalab?hl=zh-tw
https://github.com/GoogleCloudPlatform/cloudml-dist-mnist-example
cat Online\ prediction\ example.ipynb > Untitled\ Notebook.ipynb

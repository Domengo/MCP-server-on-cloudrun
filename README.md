# Zoo Guide Agent

This project implements a Zoo Tour Guide agent using the Google Agent Development Kit (ADK). The agent can answer questions about animals in a zoo by leveraging both a local database (via an MCP server) and external knowledge from Wikipedia.

## Setup

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Domengo/MCP-server-on-cloudrun.git
    cd MCP-server-on-cloudrun
    ```

2.  **Create and activate a Python virtual environment:**
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

3.  **Install the required dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Create a `.env` file:**
    Create a file named `.env` in the root of the project and add the following environment variables. You will need to replace `<your-mcp-server-url>` with the actual URL of your MCP server.

    ```
    MODEL="gemini-1.5-flash"
    MCP_SERVER_URL=<your-mcp-server-url>
    ```

## Running the Agent

This agent is designed to be deployed as a web service on Google Cloud Run. The primary way to interact with the agent is through the web UI provided after deployment.

## Deployment to Google Cloud Run

1.  **Set up your Google Cloud project:**
    ```bash
    gcloud config set project <your-gcp-project-id>
    ```

2.  **Enable the necessary Google Cloud services:**
    ```bash
    gcloud services enable \
        run.googleapis.com \
        artifactregistry.googleapis.com \
        cloudbuild.googleapis.com \
        aiplatform.googleapis.com \
        compute.googleapis.com
    ```

3.  **Set environment variables for deployment:**
    ```bash
    export PROJECT_ID=$(gcloud config get-value project)
    export REGION=$(gcloud compute project-info describe \
      --format="value(commonInstanceMetadata.items[google-compute-default-region])")
    export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")
    export SERVICE_ACCOUNT="${PROJECT_NUMBER}-compute@developer.gserviceaccount.com"
    ```

4.  **Grant IAM permissions to the service account:**
    ```bash
    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="serviceAccount:$SERVICE_ACCOUNT" \
      --role="roles/run.invoker"

    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="serviceAccount:$SERVICE_ACCOUNT" \
      --role="roles/aiplatform.user"
    ```

5.  **Deploy the agent to Cloud Run:**
    ```bash
    adk deploy cloud_run \
      --project=$PROJECT_ID \
      --region=$REGION \
      --service_name=zoo-tour-guide \
      --with_ui \
      .
    ```
    When prompted, type `Y` to allow unauthenticated invocations.

6.  **Access the agent:**
    After successful deployment, you will get a Service URL. Open this URL in your browser to interact with the Zoo Guide Agent through the ADK's web UI.

## Cleaning Up

To avoid incurring future costs, you can delete the created Cloud resources with the following commands:

```bash
gcloud run services delete zoo-tour-guide --region=$REGION --quiet
gcloud artifacts repositories delete cloud-run-source-deploy --location=$REGION --quiet
```

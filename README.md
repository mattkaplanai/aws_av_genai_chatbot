# GenAI RAG Chatbot on AWS

A document analysis chatbot built with Retrieval-Augmented Generation (RAG) on AWS. Ask questions about your documents and get accurate answers powered by Claude 3 Haiku via AWS Bedrock.

## Architecture

```
[Documents] → S3
                ↓
        Bedrock Knowledge Base
                ↓
  Titan Embeddings → OpenSearch Vector DB
                          ↓
[User] → Gradio UI → LlamaIndex Agent → RAG retrieval → Claude 3 Haiku → Answer

Docker image → ECR → App Runner (cloud deployment)
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| LLM | Claude 3 Haiku (AWS Bedrock) |
| Embeddings | Amazon Titan Embeddings v2 |
| Vector DB | Amazon OpenSearch |
| RAG Orchestration | LlamaIndex |
| Knowledge Base | AWS Bedrock Knowledge Bases |
| Frontend | Gradio |
| Storage | Amazon S3 |
| Containerization | Docker |
| Registry | AWS ECR |
| Deployment | AWS App Runner |

## Project Structure

```
.
├── agent.py          # LlamaIndex RAG agent logic
├── app.py            # Gradio chat UI
├── requirements.txt  # Python dependencies
├── .env.example      # Environment variables template
└── Dockerfile        # Container configuration
```

## Prerequisites

- Python 3.11+
- Docker
- AWS account with the following enabled:
  - Bedrock model access: **Claude 3 Haiku**
  - Bedrock model access: **Titan Embeddings v2**
- AWS CLI configured

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/kartik-nighania/av_genai_chatbot.git
cd av_genai_chatbot
```

### 2. Create virtual environment

```bash
python -m venv venv
source venv/bin/activate  # Mac/Linux
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
AWS_ACCOUNT_ID=your_account_id
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_DEFAULT_REGION=ap-south-1
BEDROCK_KNOWLEDGE_BASE_ID=your_kb_id
```

### 5. AWS Setup

1. **S3** — Create a bucket and upload your documents
2. **Bedrock Knowledge Base** — Create a KB connected to your S3 bucket
3. **OpenSearch** — Created automatically by Bedrock KB setup
4. **Sync** — Sync the data source to generate embeddings

### 6. Run locally

```bash
python app.py
```

Open `http://localhost:8080` in your browser.

## Deployment

### Build and push Docker image to ECR

```bash
# Build
docker build --platform linux/x86_64 -t av-genai-chatbot .

# Authenticate to ECR
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com

# Tag and push
docker tag av-genai-chatbot:latest \
  <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/av-genai-chatbot:latest

docker push \
  <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/av-genai-chatbot:latest
```

### Deploy with App Runner

1. Go to **AWS App Runner** in the console
2. Create a new service from ECR image
3. Set environment variables via **AWS Parameter Store**
4. Attach the IAM role with Bedrock + SSM permissions
5. Deploy — App Runner provides a public HTTPS URL

## Cleanup

To avoid ongoing AWS charges, delete resources in this order:

1. App Runner service
2. ECR repository
3. Bedrock Knowledge Base
4. OpenSearch collection
5. S3 bucket
6. Parameter Store parameters
7. IAM role

## Reference

Based on the tutorial by [Karthik Ninghana](https://github.com/kartik-nighania/av_genai_chatbot).

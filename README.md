# Serverless document chat application

This sample application allows you to ask natural language questions of any PDF document you upload. 
It combines the text generation and analysis capabilities of an LLM with a vector search of the document content. 
The solution uses serverless services such as [Amazon Bedrock](https://aws.amazon.com/bedrock/) to access foundational models, [AWS Lambda](https://aws.amazon.com/lambda/) 
to run [LangChain](https://github.com/hwchase17/langchain), and [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) for conversational memory.

## Key features

- [Amazon Bedrock](https://aws.amazon.com/de/bedrock/) for serverless embedding and inference
- [LangChain](https://github.com/hwchase17/langchain) to orchestrate a Q&A LLM chain
- [PineCone](https://www.pinecone.io) vector store
- [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) for serverless conversational memory
- [AWS Lambda](https://aws.amazon.com/lambda/) for serverless compute
- Frontend built in [React](https://react.dev/), [TypeScript](https://www.typescriptlang.org/), [TailwindCSS](https://tailwindcss.com/), and [Vite](https://vitejs.dev/).
- Run locally or deploy to [AWS Amplify Hosting](https://aws.amazon.com/amplify/hosting/)
- [Amazon Cognito](https://aws.amazon.com/cognito/) for authentication

## How the application works

1. Uploads a PDF document into an [Amazon S3](https://aws.amazon.com/s3/) using pre-signed URL 
2. The new document model is created in DynamoDB (For Metadata and saving conversations)
3. Post that metadata extraction & document embedding takes place. 
4. The process converts the text in the document into vectors. The vectors are loaded into a vector index and stored in Pinecone DB for later use.
5. When a user chats with a PDF document and sends a prompt to the backend, a Lambda function retrieves the index from Pinecone and searches for information related to the prompt.
6. A LLM then uses the results of this vector search, previous messages in the conversation to formulate a response to the user.

## Deployment instructions

### Prerequisites

- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
- [NodeJS](https://nodejs.org/en) 20.x or greater


### Amazon Bedrock setup

This application can be used with a variety of LLMs via Amazon Bedrock. See [Supported models in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-service.html#models-supported) for a complete list.

By default, this application uses amazon.titan-text-express-v1 to generate embeddings and responses.

### Pinecone

Create a [pinecone dashboard](http://pinecone.io), copy and paste the following details in constants file depending on the index name

```bash
    API_KEY : "febe9883-869a-4c4e-bb8c-356a5b356f00",
    PINECONE_ENV : "gcp-starter",
    PINECONE_INDEX_NAME : "industrility-index",
```

### Deploy the application with AWS SAM

1. Change to the `backend` directory and [build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html) the application:

   ```bash
   cd backend
   sam build
   ```
2. Setup credentials using AWS Identity and Access Management 

3. [Deploy](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html) the application into your AWS account:

   ```bash
   sam deploy --guided
   ```

4. For **Stack Name**, choose `serverless-pdf-chat`.

5. For the remaining options, keep the defaults by pressing the enter key.

AWS SAM will now provision the AWS resources defined in the `backend/template.yaml` template. Once the deployment is completed successfully, you will see a set of output values similar to the following:

```bash
CloudFormation outputs from deployed stack
-------------------------------------------------------------------------------
Outputs
-------------------------------------------------------------------------------
Key                 CognitoUserPool
Description         -
Value               us-east-1_gxKtRocFs

Key                 CognitoUserPoolClient
Description         -
Value               874ghcej99f8iuo0lgdpbrmi76k

Key                 ApiGatewayBaseUrl
Description         -
Value               https://abcd1234.execute-api.us-east-1.amazonaws.com/dev/
-------------------------------------------------------------------------------
```

You can find the same outputs in the `Outputs` tab of the `serverless-pdf-chat` stack in the AWS CloudFormation console. In the next section, you will use these outputs to run the React frontend locally and connect to the deployed resources in AWS.

### Run the React frontend locally

Create a file named `.env.development` in the `frontend` directory. [Vite will use this file](https://vitejs.dev/guide/env-and-mode.html) to set up environment variables when we run the application locally.

Copy the following file content and replace the values with the outputs provided by AWS SAM: (You can keep them as it is, as the backend stack is up and running)

```plaintext
VITE_REGION=us-east-1
VITE_API_ENDPOINT=https://abcd1234.execute-api.us-east-1.amazonaws.com/dev/
VITE_USER_POOL_ID=us-east-1_gxKtRocFs
VITE_USER_POOL_CLIENT_ID=874ghcej99f8iuo0lgdpbrmi76k
```

Next, install the frontend's dependencies by running the following command in the `frontend` directory:

```bash
npm ci
```

Finally, to start the application locally, run the following command in the `frontend` directory:

```bash
npm run dev
```

Vite will now start the application under `http://localhost:5173`. As the application uses Amazon Cognito for authentication, you will be greeted by a login screen. In the next step, you will create a user to access the application.
github-actions-with-aws-sam
Commands to install Docker in Amazon Linux 3

sudo yum update -y
sudo yum search docker -y
sudo yum install docker -y
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo systemctl status docker.service

Install git

sudo yum install git -y

Install SAM CLI in linux
Run the following commands to install SAM CLI. X86_64

wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
sam --version

Creating the AWS SAM application

sam init -r python3.8 -n github-actions-with-aws-sam --app-template "hello-world"

How to test the SAM app in local?

Go to the folder that you've created the SAM application and run the following commands:

sam local invoke HelloWorldFunction -e events/event.json

The function response is: {"message": "hello world"}

Test the API Gateway functionality in front of the Lambda function by first starting the API locally:

sam local start-api

Use curl to call the hello API:

curl http://127.0.0.1:3000/hello

The API response should be: {"message": "hello world"}
To setup GitHub Actions
GitHub Secrets

Create 2 GitHub secrets in your repository with following:

    AWS_ACCESS_KEY_ID 
    AWS_SECRET_ACCESS_KEY

Create Pipeline file

Create a new folder as .github/workflows in the root path of the cloned repo.

mkdir -p .github/workflows

Create a new file called sam-pipeline.yml under the .github/workflows directory.

vi .github/workflows/sam-pipeline.yml

Add these lines into that file:

on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v2
      - uses: aws-actions/setup-sam@v2
        with:
          use-installer: true
          token: ${{ secrets.GITHUB_TOKEN }}
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ###REGION###
      # Build inside Docker containers
      - run: sam build --use-container
      # Prevent prompts and failure when the stack is unchanged
      - run: sam deploy --no-confirm-changeset --no-fail-on-empty-changeset

Replace ###REGION### with your AWS Region.
Deploying your application

Add all the files to your git repository, commit the changes, and push to GitHub.

git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/<git-user-name>/github-actions-with-aws-sam.git
git push -u origin main

Replace the git-user-name with your GitHub account's user name or organization's name where. you have your repository.
Testing the application

Use curl to test the API:

curl https://<api-id>.execute-api.us-east-1.amazonaws.com/Prod/hello/

The API response should be: {"message": "hello world"}
Cleanup

Run aws configure to configure aws cli in your insntance.

To delete the sample application that you created, use the AWS CLI. Assuming you used your project name for the stack name, you can run the following:

sam delete --stack-name "github-actions-with-aws-sam" --region ###region###

Replace the ###region### with your region name where you have created your bucket
Resources

See the AWS SAM developer guide for an introduction to SAM specification, the SAM CLI, and serverless application concepts.

Next, you can use AWS Serverless Application Repository to deploy ready to use Apps that go beyond hello world samples and learn how authors developed their applications: AWS Serverless Application Repository main page

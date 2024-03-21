![1_bMePcyU92XDFL-QozSplXg](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/64a0924e-c695-4715-9e99-b05fe2124d33)

# AWS Lambda and GitHub Actions

This guide explains how to set up a GitHub Actions workflow for deploying an AWS Lambda function.

## Overview

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers. GitHub Actions is a feature on GitHub that helps automate workflows in your repository.

## Prerequisites

- An AWS account with Lambda access
- A GitHub repository containing the code for your Lambda function
- Basic understanding of YAML syntax
- Familiarity with setting up GitHub Actions workflows

## Workflow
**Step 1: Create a Lambda Function on AWS**

1. Go to the Lambda page inside AWS by searching for Lambda.
   
![aws-lambda](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/9811815a-2f56-4a04-ba3d-7d1a9c494186)
   
2. Create a new function by clicking on the "Create function" button.
   
![aws-lambda-create-function](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/0113903a-7dad-4724-8410-eaf70c0335c7)

3. Give the function a name, such as "my-function", and use the defaults for other settings.
   
4. After creating the function, you'll be redirected to the console page for your Lambda function.
   
![aws-lambda-code](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/bd1f13a7-3a7f-44c0-81ad-ff3ed41952b5)

You can test the function by pressing the test button in the upper right. This is the only way to run the function right now.

**Step 2: Optional - Add a Trigger**

1. If you want to trigger the function, such as through an API gateway, you can add a trigger.
![1_FjYuTA7v-x3tN_yfJYXvTQ](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/a3507f7b-7452-4b2b-98fa-8d99cc935c32)
![1_fRHnyBBbhB-lhc8-GvthBQ](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/5768fc0c-6243-40dc-a1a8-3069bb896b55)

2.Find The Endpoint and Test the End Point

**Step 3: Edit the Function Code**

1. Scroll down on the Lambda function page to view the code for your function. It's typically a hello-world application.
2. You can edit the code directly on this page. However, for non-trivial functions, it's recommended to use versioning with Git.

**README: Deploying Lambda Function Using GitHub Actions**

**Step 1: Create GitHub Repo**

1. Go to GitHub and create a new repository for your Lambda function.
2. Name it appropriately, such as "my-lambda-function".

**Step 2: Set Up Node Project**

1. Create a new directory for your project and navigate into it.
```
mkdir my-function
cd my-function
```
2. Initialize the project with npm.
```
npm init -y
```
3. Create a JavaScript file named `index.js` and add the following code:
```javascript
exports.handler = async (event) => {
  const response = {
    statusCode: 200,
    body: JSON.stringify("Hello from Lambda and Github!"),
  }
  return response
}
```

**Step 3: Deploy Lambda Using GitHub Actions**

1. Go to the Actions tab in your GitHub repository.
   
![Untitled design (1)](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/7755c570-956d-4cbc-a386-f79281f1d12d)

2. Click on "Set up a workflow yourself".
   
![start-githubaction-workflow](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/b70b6bc4-65e5-425f-a7ac-c78b2b4d9cad)

3. Replace the template code with the following YAML snippet:

```yaml
name: deploy to lambda
on: [push]
jobs:
  deploy_source:
    name: build and deploy lambda
    strategy:
      matrix:
        node-version: [12.x]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: npm install and build
        run: |
          npm ci
          npm run build --if-present
        env:
          CI: true
      - name: zip
        uses: montudor/action-zip@v0.1.0
        with:
          args: zip -qq -r ./bundle.zip ./
      - name: default deploy
        uses: appleboy/lambda-action@master
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws_region: eu-west-1
          function_name: my-function
          zip_file: bundle.zip
```

The `.yml` file contains the configuration for GitHub Actions, specifying the workflow that automates the deployment of your Lambda function. Here's an explanation of each section:
    
1. **Name**: Defines the name of this action, which helps identify the workflow. In this case, it's named "deploy to lambda".
    
2. **On**: Specifies the trigger for the workflow. The `[push]` event indicates that the workflow will run anytime you push code to the repository.
    
3. **Jobs**: Defines the job to be executed. In this case, there is only one job called "build and deploy lambda".
    
4. **Strategy**: Specifies the configuration matrix, which includes the Node.js version to use. Here, it's set to `node-version: [12.x]`.
    
5. **Runs-on**: Indicates the type of runner environment for the job. Here, it runs on the latest version of Ubuntu.
    
6. **Steps**: Contains a list of steps to be executed within the job.
    
   a. **actions/checkout@v1**: This step checks out the code from the GitHub repository, allowing subsequent steps to access the codebase.
        
   b. **Use Node.js ${{ matrix.node-version }}**: Installs the specified Node.js version on the Ubuntu machine.
        
   c. **npm install and build**: Executes `npm ci` to install dependencies and `npm run build` to build the project. Useful for managing NPM dependencies in the Lambda function.
        
   d. **zip**: Compresses all files into a zip file named `bundle.zip`.
        
   e. **default deploy**: Deploys the `bundle.zip` file to the Lambda function named "my-function". Modify the `function_name` parameter if you named your Lambda differently.

4. Save the file and commit it to your GitHub repository.
   

**Step 4: Add AWS Access Keys to GitHub Secrets**

1. Go to the IAM page in your AWS account.
2. Navigate to "Users" and select your user.
3. Go to "Security credentials" and either use existing access keys or create a new one.
4. Add the access key ID and secret access key to your GitHub repository secrets.
![github-secrets](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/bcff5be4-4057-4701-af90-739870ac6630)

**Step 5: Trigger Deployment**

1. Commit and push any changes to your GitHub repository.
2. Go to the "Actions" tab in your GitHub repository to monitor the deployment process.

![Your paragraph text](https://github.com/RAFAT-DEVELOPER/aws-lambda-github-actions/assets/8677005/fcbcfa54-930e-4ba8-b5fb-b96ced5dcf6d)

4. Once deployed, refresh the AWS Lambda console to see the updated code.

Now your Lambda function will be automatically deployed from your GitHub repository every time you push new code.


## Conclusion

By setting up a GitHub Actions workflow, you can automate the deployment of your AWS Lambda functions, streamlining your development process and enabling continuous integration and delivery.

# Bird-Watching & Buried Marks CI/CD Pipelines

This repository contains the Jenkins pipelines (`Jenkinsfile`) used to automate the infrastructure provisioning (Terraform) and application deployment for the Bird-Watching and Buried Marks applications.

# How to Run the Pipeline
The pipeline is fully parameterized. When you trigger a build in Jenkins, you will be prompted to select the following:

- Environment(dev, stage or prod)
- Action:
    -  plan - generates and displays executive plan
    - apply - Applies changes to reach the desired state (requires manual approval for prod)
    - destroy - Destroys the managed infrastructure (generates a destroy plan and requires manual approval)

## Prerequisites

For security reasons, our Jenkins server is deployed in a private subnet and is not exposed to the public internet. We use AWS Systems Manager (SSM) Port Forwarding to securely access the Jenkins UI.

Before you begin, ensure you have the following installed on your local machine:

[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
[AWS Session Manager Plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

## Accessing the Jenkins Dashboard

To establish a secure tunnel to the Jenkins EC2 instance, open your terminal and run the following command:

aws ssm start-session \
    --target Instance_id \
    --document-name AWS-StartPortForwardingSession \
    --parameters '{"portNumber":["8080"],"localPortNumber":["8080"]}'

pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stage', 'prod'],
            description: 'Choose environment'
        )
        choice(
            name: 'ACTION',
            choices: ['plan', 'apply', 'destroy'],
            description: 'Terraform action'
        )
    }

    environment {
        TF_DIR = "terraform-2apps/environments/${params.ENVIRONMENT}"
        AWS_REGION = 'eu-north-1'
    }

    stages {
        stage('Check Tools') {
            steps {
                sh 'terraform version'
                sh 'aws --version'
            }
        }

        stage('Checkout Terraform') {
            steps {
                git(
                    url: 'https://github.com/INITOPS-TEAM/terraform-bird-watching-buried-marks.git',
                    branch: 'main'
                )
            }
        }

        stage('Terraform Init') {
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform init'
                }
            }
        }	

	stage('Terraform Plan') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'ssh-public-key-file',
                        variable: 'SSH_KEY_PATH'
                    )
                ]) {
                    dir("${TF_DIR}") {
                        script {
                            if (params.ACTION == 'destroy') {
                                sh """
                                    terraform plan -destroy \
                                        -target=module.birdwatching.aws_instance.lb \
                                        -target=module.birdwatching.aws_instance.app \
                                        -target=module.birdwatching.aws_instance.db \
                                        -target=module.birdwatching.aws_instance.consul \
                                        -target=module.eks \
                                        -var="public_key_path=${SSH_KEY_PATH}" \
                                        -out=tfplan
                                """
                            } else {
                                sh """
                                    terraform plan \
                                        -var="public_key_path=${SSH_KEY_PATH}" \
                                        -out=tfplan
                                """
                            }
                        }
                    }
                }
            }
        }

	stage('Approval') {
            when {
                expression { return params.ACTION == 'apply' || params.ACTION == 'destroy' }
            }
            steps {
                script {
                    input message: "Review the Terraform Plan. Do you approve the ${params.ACTION} for ${params.ENVIRONMENT} environment?", ok: "Approve"
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression { return params.ACTION == 'apply' }
            }
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform apply tfplan'
                }
            }
        }

	stage('Deploy Landing Page') {
            when {
                expression { return params.ACTION == 'apply' }
            }
            steps {
                script {
                    dir("${TF_DIR}") {
                        env.S3_BUCKET = sh(script: 'terraform output -raw landing_s3_bucket_name', returnStdout: true).trim()
                        env.CF_DIST_ID = sh(script: 'terraform output -raw landing_cloudfront_id', returnStdout: true).trim()
                    }

                   dir('landing-workspace') {
                        git branch: 'main', url: 'https://github.com/INITOPS-TEAM/landing-page-bird-watching-buried-marks.git'

                        sh """
                            echo "Deploying dir landing-page/ to ${env.S3_BUCKET}..."
                            aws s3 sync landing-page/ s3://${env.S3_BUCKET} --delete
                            aws cloudfront create-invalidation --distribution-id ${env.CF_DIST_ID} --paths "/*"
                        """
                    } 
                }
            }
        } 

        stage('Terraform Destroy') {
            when {
                expression { return params.ACTION == 'destroy' }
            }
            steps {
                dir("${TF_DIR}") {
                    sh 'terraform apply tfplan'
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline ${params.ACTION} completed successfully for ${params.ENVIRONMENT}"
        }
        failure {
            echo "Pipeline ${params.ACTION} FAILED for ${params.ENVIRONMENT}"
        }
        always {
            cleanWs()
        }
    }
}

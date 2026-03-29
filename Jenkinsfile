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

pipeline {

    agent {
        label 'Java'
    }

    environment {
        AWS_REGION = 'ap-south-1'
        IMAGE_NAME = '832569409044.dkr.ecr.ap-south-1.amazonaws.com/petclinic-java-project'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

                git branch: 'main',
                    url: 'https://github.com/geetha-5656/java-project2.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'

                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running JUnit unit tests...'

                sh 'mvn test -Dtest="!MySqlIntegrationTests"'
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

        stage('Upload Artifact to S3') {
            steps {
                sh """
                    aws s3 cp target/*.jar \
                    s3://every-build-artifacts/petclinic/build-${BUILD_NUMBER}.jar
                """
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin \
                    832569409044.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh """
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
    }

    post {

        success {
            echo 'Application image successfully built and pushed to ECR.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            cleanWs()
        }
    }
}

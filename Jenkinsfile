pipeline {

    agent {
        label 'Java'
    }

    environment {
        AWS_REGION = 'ap-south-1'
        IMAGE_NAME = '351245513944.dkr.ecr.ap-south-1.amazonaws.com/petclinic-java-project'
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
        echo 'Running fast unit tests...'

        sh '''
            mvn -B test \
                -Dtest='*Test,!*IntegrationTest,!*IntegrationTests,!MySqlIntegrationTests,!CrashControllerIntegrationTests' \
                -DfailIfNoTests=false
        '''
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'target/surefire-reports/*.xml'
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
                    echo "Building Docker image..."

                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    docker tag \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh """
                    echo "Logging in to private ECR..."

                    aws ecr get-login-password \
                        --region ${AWS_REGION} | \
                    docker login \
                        --username AWS \
                        --password-stdin \
                        351245513944.dkr.ecr.ap-south-1.amazonaws.com
                """
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh """
                    echo "Pushing image to ECR..."

                    docker push ${IMAGE_NAME}:${IMAGE_TAG}

                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
    }

    post {

        success {
            echo '=========================================='
            echo 'Pipeline completed successfully!'
            echo 'Application image built and pushed to ECR.'
            echo '=========================================='
        }

        failure {
            echo '=========================================='
            echo 'Pipeline failed.'
            echo 'Check the failed stage in the Jenkins console.'
            echo '=========================================='
        }

        always {
            cleanWs()
        }
    }
}

       

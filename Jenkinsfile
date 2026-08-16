pipeline {

    agent {
        label 'Java'
    }

    environment {
        
AWS_REGION = 'ap-south-1'
        IMAGE_NAME = '832569409044.dkr.ecr.ap-south-1.amazonaws.com/petclinic-java-project'
        IMAGE_TAG = "${BUILD_NUMBER}"

        EC2_HOST = '13.201.204.209'
        EC2_USER = 'ubuntu'
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
        sh 'mvn test -Dtest="**/*Tests" -DfailIfNoTests=false'
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
    }
}

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Upload Artifact to S3') {
            steps {
                sh """
                aws s3 cp target/*.jar s3://every-build-artifacts/petclinic/build-${BUILD_NUMBER}.jar
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
                docker login --username AWS --password-stdin 832569409044.dkr.ecr.ap-south-1.amazonaws.com
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

       stage('Deploy to EC2') {
    steps {
        sshagent(credentials: ['application-server']) {
            sh """
ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin 832569409044.dkr.ecr.ap-south-1.amazonaws.com

docker pull ${IMAGE_NAME}:latest

docker stop petclinic || true
docker rm petclinic || true

docker run -d --name petclinic \
-p 8080:8080 \
--restart always \
${IMAGE_NAME}:latest
EOF
"""
        }
    }
}

        stage('Health Check') {
            steps {
                sh """
                sleep 30
                curl http://${EC2_HOST}:8080
                """
            }
        }
    }

    post {

        success {
            echo 'Application deployed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            cleanWs()
        }
    }
}



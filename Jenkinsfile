pipeline{
       agent{
           label 'Java'
           }
        environment{
           IMAGE_NAME = "dockerpracticelab/petclinic"
           IMAGE_TAG = "${BUILD_NUMBER}"


           EC2_HOST = "13.201.122.43"
           EC2_USER = "ubuntu"
           }

           stages {
               stage ('Checkout'){
                steps{
                  echo "Checking out source code.."
                  git branch: 'main',
                      url: 'https://github.com/geetha-5656/java-project2.git'
                  }
            }
            
        stage('Build') {
            steps {
                echo "Building application..."
                sh 'mvn clean package'
            }
        }


        stage('Unit Test') {
            steps {
                echo "Running unit tests..."
                sh 'mvn test'
            }
 
}

         stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
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

        stage('Docker Login') {
            steps {
            withCredentials([usernamePassword(
            credentialsId: 'dockerhub',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            '''
        }
    }
}
     stage('Push Docker Image') {
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
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                    docker pull ${IMAGE_NAME}:latest
                    docker stop petclinic || true
                    docker rm petclinic || true
                    docker run -d --name petclinic -p 8080:8080 --restart always ${IMAGE_NAME}:latest
                    '
                    """
                }
            }
        }


      stage('Health Check') {
            steps {
                sh """
                sleep 20
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

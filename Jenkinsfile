pipeline {
	agent any

	stages{

		stage('Build'){
                       steps{
                         sh 'mvn clean package -DskipTests'
			}
                }

                stage('Test'){
                      steps{
                        sh 'mvn test -DskipTests'
                       }
                }

                stage('Archive Artifact'){
                        steps{
                          archiveArtifacts artifacts: 'target/*.jar'
                        }
                }
                stage('Docker build'){
                       steps {
                          sh 'docker build -t petclinic .'
                       }
                }
                stage('Run Container'){
                       steps{
                           sh '''
                           docker stop petclinic || true
                           docker rm petclinic || true
                           docker run -d --name petclinic -p 8080:8080 petclinic '''
                        }
                 }

         }
}

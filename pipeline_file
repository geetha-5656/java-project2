pipeline {
	agent any

	stages{
		
		stage('Checkout'){
			steps{
			   git 'https://github.com/geetha-5656/java-project2.git'
		         }
                }

		stage('Build'){
                       steps{
                         sh 'mvn clean package -DskipTests'
			}
                }

                stage('Test'){
                      steps{
                        sh 'mvn test'
                       }
                }
                
		stage('Deploy'){
                      steps{
                        sh '''
                        pkill -f 'java -jar' || true
                        nohup java -jar target/*.jar > app.log 2>&1 &
                        '''
                       }
                }
         }
}

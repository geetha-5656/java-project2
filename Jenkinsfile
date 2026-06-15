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

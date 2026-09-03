pipeline{
    agent none
    stages{
        stage('Maven Compile'){
            agent{
                label 'maven-cloud-node'
            }
            steps{
                sh 'mvn compile'
            }
        }
        stage('SCA'){
            agent{
                docker {
                    image 'owasp/depedency-check:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock --entrypoint=' 
                }
            }
            steps{
                sh '/usr/share/depedency-check.sh --scan . --project "VulnerableJavaWebApplication" --format ALL'
                archiveArtifacts artifacts: 'depedency-check-report.html'
                archiveArtifacts artifacts: 'depedency-check-report.json'
                archiveArtifacts artifacts: 'depedency-check-report.xml'
            }
        }
        stage('Build Docker Image'){
            agent{
                docker {
                    image 'docker:dind'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock' 
                }
            }
            steps{
                sh 'docker build -t vulnerable-java-application:0.1 .'
            }
        }
        stage('Run Docker Image'){
            agent{
                label 'built-in'
            }
            steps{
                sh 'docker rm --force vulnerable-java-application'
                sh 'docker run --name vulnerable-java-application -p 9000:9000 -d vulnerable-java-application:0.1'
            }
        }
    }
}

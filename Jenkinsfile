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
        stage('SCA (Trivy)'){
            agent{
                docker {
                    image 'aquasec/trivy:latest'
                    args '-u root --entrypoint='
                }
            }
            steps{
                // Scan direktori/dependensi aplikasi dan simpan laporan
                sh 'trivy fs --offline-scan --format table -o trivy-report.txt .'
                sh 'trivy fs --offline-scan --format json -o trivy-report.json .'                
                
                archiveArtifacts artifacts: 'trivy-report.txt'
                archiveArtifacts artifacts: 'trivy-report.json'
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

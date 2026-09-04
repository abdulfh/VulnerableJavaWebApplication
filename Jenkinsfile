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
        stage('SCA (Trivy Image Scan)') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                // 1. Tampilkan hasil scan di Console Log
                sh 'trivy image --offline-scan vulnerable-java-application:0.1'

                // 2. Download template HTML resmi Trivy
                sh 'wget -q https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O html.tpl'

                // 3. Generate laporan HTML menggunakan html.tpl lokal
                sh 'trivy image --offline-scan --format template --template "@html.tpl" -o trivy-report.html vulnerable-java-application:0.1'
                
                // 4. Simpan laporan di Jenkins Artifacts
                archiveArtifacts artifacts: 'trivy-report.html'
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

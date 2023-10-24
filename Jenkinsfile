pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/mukeshr-29/netflix-project-11.git'
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'sonar') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=netflix-project \
                        -Dsonar.projectKey=netflix-project  '''
                    }
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
    }
    post {
        always {
            script {
                emailext attachLog: true,
                    subject: "'${currentBuild.result}'",
                    body: "Project: ${env.JOB_NAME}<br/>" +
                        "Build Number: ${env.BUILD_NUMBER}<br/>" +
                        "URL: ${env.BUILD_URL}<br/>",
                    to: 'rmukeshh29@gmail.com',
                    attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
                
            }
        }
        stage("docker build and push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh "docker build --build-arg TMDB_V3_API_KEY=0fca49232455b7eb9992e977753c7b14 -t netflix-project ."
                        sh "docker tag netflix-project mukeshr29/netflix-project:latest"
                        sh "docker push mukeshr29/netflix-project:latest"
                    }
                }
            }
        }
        stage("trivy-scan"){
            steps{
                sh "trivy image mukeshr29/netflix-project:latest > trivy.txt"
            }
        }
    }
}



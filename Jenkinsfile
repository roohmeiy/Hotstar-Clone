pipeline {
   agent any
   tools {
       jdk 'jdk17'
       nodejs 'node20'
   }
   environment {
       SCANNER_HOME = tool 'sonar-scanner'
   }
   stages {
       stage('Repo Clone') {
           steps {
               echo "git clone......."
               git url: 'https://github.com/roohmeiy/Hotstar-Clone.git', branch: 'main'
           }
       }
       
       stage("Sonarqube Analysis") {
           steps {
               withSonarQubeEnv('sonar-server') {
                   sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Hotstar -Dsonar.projectKey=Hotstar -X"
               }
           }
       }
       
       stage("Quality Gate") {
           steps {
               script {
                   waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
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
       
       stage('Docker Scout FS') {
           steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                       sh 'docker-scout quickview fs://.'
                       sh 'docker-scout cves fs://.'
                   }
               }
           }
       }
       
       stage("Docker Build & Push") {
           steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                       sh "docker build -t hotstar ."
                       sh "docker tag hotstar roohmeiy/hotstar:latest"
                       sh "docker push roohmeiy/hotstar:latest"
                   }
               }
           }
       }
       
       stage('Docker Scout Image') {
           steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                       sh 'docker-scout quickview roohmeiy/hotstar:latest'
                       sh 'docker-scout cves roohmeiy/hotstar:latest'
                       sh 'docker-scout recommendations roohmeiy/hotstar:latest'
                   }
               }
           }
       }
       
       stage('Deploy to kubernets') {
           steps {
               script {
                   dir('K8S') {
                       withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                           sh 'kubectl apply -f deployment.yml'
                           sh 'kubectl apply -f service.yml'
                       }
                   }
               }
           }
       }
   }
}

pipeline {
    agent {
        kubernetes {
            label 'maven-agent'
            defaultContainer 'maven'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9.8-ibmjava-8
    command:
    - cat
    tty: true
    resources:
      limits:
        memory: "2Gi"
        cpu: "1000m"
      requests:
        memory: "1Gi"
        cpu: "500m"
    volumeMounts:
    - name: maven-repo
      mountPath: /root/.m2
  volumes:
  - name: maven-repo
    emptyDir: {}
"""
        }
    }
    stages {
        stage('Build Artifact') {
            steps {
                container('maven') {
                    sh "mvn clean package -DskipTests=true"
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }
        stage('Run Tests') {
            steps {
                container('maven') {
                    sh "mvn test"
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                container('maven') {
                    withSonarQubeEnv('SonarQube') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                container('maven') {
                    // Replace with your deployment script or command
                    sh "echo Deploying to staging environment"
                }
            }
        }
    }
    post {
        failure {
            mail to: 'team@example.com',
                subject: "Build failed in Jenkins: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Something is wrong with ${env.JOB_NAME} #${env.BUILD_NUMBER}:\nCheck the Jenkins job here: ${env.BUILD_URL}"
        }
    }
}

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
"""
        }
    }
    stages {
        stage('Build Artifact') {
            steps {
                container('maven') {
                    sh "mvn clean package -DskipTests=true"
                }
              post {
                always {
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec'
                }
              }
            }
        }
    }
}

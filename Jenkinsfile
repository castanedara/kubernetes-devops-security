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
        }
        stage('Run Tests') {
            steps {
                container('maven') {
                    sh "mvn test"
                }
            }
        }
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
        }
    }
}

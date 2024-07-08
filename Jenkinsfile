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
  - name: docker
    image: docker:25.0.5-cli
    command: 
      - cat
    tty: true
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
        stage('Docker Build and Push') {
            steps {
                container('docker') {    
                    withDockerRegistry([credentialsId: "nexus-docker", url: "docker-desarrollo-royaltechnology.chickenkiller.com"]) {
                    sh 'printenv'
                    sh 'docker build -t docker-desarrollo-royaltechnology.chickenkiller.com/numeric-app:""$GIT_COMMIT"" .'
                    sh 'docker push docker-desarrollo-royaltechnology.chickenkiller.com/numeric-app:""$GIT_COMMIT""'
                    }
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
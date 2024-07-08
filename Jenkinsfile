pipeline {
    agent {
        kubernetes {
            label 'maven-agent'
            defaultContainer 'kubectl'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - cat
    tty: true
    volumeMounts:
      - name: kubeconfig
        mountPath: /root/.kube
  - name: git
    image: alpine/git:latest
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true
  - name: docker
    image: docker:25.0.5-cli
    command: 
      - cat
    tty: true
    volumeMounts:
      - name: docker-socket
        mountPath: /var/run/docker.sock
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
  - name: docker-socket
    hostPath:
      path: /var/run/docker.sock
  - name: kubeconfig
    secret:
      secretName: kubeconfig
"""
        }
    }

    environment {
        DOCKER_REGISTRY = 'docker-desarrollo-royaltechnology.chickenkiller.com'
    }

    stages {
        stage('Checkout code') {
            steps {
                echo "Cambios en la rama ${env.BRANCH_NAME}\n"
                echo 'Checkout code'
                checkout scm
            }
        }


        stage('Set up environment') {
            steps {
                container('git') {
                    script {
                        // abortAllPreviousBuildInProgress(currentBuild)

                        env.WORKSPACE_GIT = sh(returnStdout: true, script: "pwd").trim()
                        echo "Git WORKSPACE_GIT: ${env.WORKSPACE_GIT}"
                        sh "git config --global --add safe.directory ${env.WORKSPACE_GIT}"    

                        env.GIT_URL = sh(returnStdout: true, script: "git config --get remote.origin.url").trim()
                        env.GIT_REPO_NAME = "${env.GIT_URL}".tokenize('/').last().split("\\.")[0]
                        env.GIT_SHORT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
                        env.GIT_COMMIT = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%H'").trim()
                        env.GIT_COMMIT_MSG = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%s'").trim()

                        if (env.CHANGE_ID == null) {
                            env.GIT_USER_NAME = sh(script: "git show -s --pretty='%an' ${env.GIT_COMMIT}", returnStdout: true).trim()
                            env.GIT_USER_EMAIL = sh(script: "git --no-pager show -s --format='%ae' ${env.GIT_COMMIT}", returnStdout: true).trim()
                        }
                        echo "Usuario: ${env.GIT_USER_NAME}"
                        echo "Email: ${env.GIT_USER_EMAIL}"
                    }
                }
            }
        }

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
                    withCredentials([usernamePassword(credentialsId: 'nexus-docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // No imprimimos printenv para evitar exponer variables sensibles
                        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS https://${DOCKER_REGISTRY}'
                        sh 'docker build -t ${DOCKER_REGISTRY}/numeric-app:${GIT_COMMIT} .'
                        sh 'docker push ${DOCKER_REGISTRY}/numeric-app:${GIT_COMMIT}'
                    }
                }
            }
        }

         stage('Kubernetes Deployment - DEV') {
            steps {
                container('kubectl') {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh "cd ${env.WORKSPACE_GIT}"
                        sh "sed -i 's#replace#${DOCKER_REGISTRY}/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                        sh "kubectl apply -f k8s_deployment_service.yaml"
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

// Definición de la función abortAllPreviousBuildInProgress

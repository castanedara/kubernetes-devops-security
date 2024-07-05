node('maven-agent') {
    podTemplate(yaml: """
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
""") {
        node(POD_LABEL) {
            try {
                stage('Build Artifact') {
                    container('maven') {
                        sh "mvn clean package -DskipTests=true"
                    }
                }

                stage('Run Tests') {
                    container('maven') {
                        sh "mvn test "
                    }
                }
            } finally {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
            }
        }
    }
}

def label = "maven-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'maven', image: 'maven:3-jdk-8', ttyEnabled: true, command: 'cat')
  ])
{
    node(label) {
        container('maven') {
            stage('Checkout') {
                checkout scm
            }
            ansiColor('xterm') {
                // TODO Add persistentVolumeClaim to greatly speed this up
                stage('Maven install') {
                  sh 'mvn install -Dmaven.test.skip=true -s .mvn/settings.xml'
                }
                stage('Integration Test') {
                  sh 'mvn integration-test'
                }
                stage('Deploy snapshot') {
                    withCredentials([file(credentialsId: 'monplat-jenkins-json', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'mvn deploy -s .mvn/settings.xml'
                    }
                }
            }
        }
    }
}

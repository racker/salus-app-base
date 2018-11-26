def label = "maven-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'maven', image: 'maven:3-jdk-8', ttyEnabled: true, command: 'cat')
  ])
{
    environment {
        GOOGLE_APPLICATION_FILE = credentials('monplat-jenkins-json')
    }
    node(label) {
        container('maven') {
            stage('Checkout') {
                checkout scm
            }
            ansiColor('xterm') {
                // TODO Add persistentVolumeClaim to greatly speed this up
                stage('Maven install') {
                  sh 'export GOOGLE_APPLICATION_CREDENTIALS=$GOOGLE_APPLICATION_FILE'
                  sh 'mvn install -Dmaven.test.skip=true'
                }
                stage('Integration Test') {
                  sh 'mvn integration-test'
                }
                stage('Deploy snapshot') {
                  sh 'mvn deploy'
                }
            }
        }
    }
}

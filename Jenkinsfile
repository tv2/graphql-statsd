@Library(['commonLibrary', 'playbeLibrary']) _
node {
  try {
    stage('Checkout') {
      deleteDir()
      checkout scm
    }

    def nodeBuilder = docker.image('node:9.11.1-slim')

    nodeBuilder.pull()
    nodeBuilder.inside() {
      stage('Build') {
        sh 'sudo npm install'
      }

      stage('Unit Test') {
        sh 'sudo npm run test'
      }

      stage('Publish to Artifactory') {
        withCredentials([usernamePassword(credentialsId: 'ci-arti-saas', usernameVariable: 'userVariable', passwordVariable: 'passwordVariable')]) {
          // set URL to registry and publish with credentials
          // npm set login
          // npm publish --registry 'https://tv2.jfrog.io/tv2/api/npm/npm-local/'
          sh 'sudo npm config set username $userVariable'
          sh 'sudo npm config set password $passwordVariable'
          sh 'sudo npm publish --registry "https://tv2.jfrog.io/tv2/api/npm/npm-local"'
        }
      }
    }

  } catch(Exception e) {
    sendNotificationFailed('jems', e)
  }
}
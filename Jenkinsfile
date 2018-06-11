@Library(['commonLibrary', 'playbeLibrary']) _
node {
  try {
    stage('Checkout') {
      deleteDir()
      checkout scm
    }

    def nodeBuilder = docker.image('tv2-devops-docker-production.jfrog.io/buildtools-node-builder:latest') //

    //https://tv2.jfrog.io/tv2/devops-docker-production/buildtools-node-builder

    nodeBuilder.pull()
    nodeBuilder.inside() {
      stage('Build') {
        sh 'npm install'
      }

      stage('Unit Test') {
        //sh 'npm run test'
      }

      stage('Publish to Artifactory') {
        withCredentials([usernamePassword(credentialsId: 'ci-arti-saas', usernameVariable: 'userVariable', passwordVariable: 'passwordVariable')]) {
          // set URL to registry and publish with credentials
          // npm set login
          // npm publish --registry 'https://tv2.jfrog.io/tv2/api/npm/npm-local/'
          sh 'echo > "_auth = ${userVariable}:${password}"\n email = playbackend@tv2.dk \n always-auth = true"'
          //sh 'npm config set username $userVariable'
          //sh 'npm config set password $passwordVariable'
          sh 'npm publish --registry "https://tv2.jfrog.io/tv2/api/npm/npm-local"'
        }
      }
    }

  } catch(Exception e) {
    sendNotificationFailed('jems', e)
  }
}
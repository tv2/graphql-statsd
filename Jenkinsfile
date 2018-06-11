@Library(['commonLibrary', 'playbeLibrary']) _
node {
  try {
    stage('Checkout') {
      deleteDir()
      checkout scm
    }

    def buildProps = readProperties file: 'build.properties'

    conf = [
      "major": "${buildProps['major']}"
      "minor": "${buildProps['minor']}"
    ]

    def releaseName = conf['version']

    stage('Set Version') {
      def packageJson = readJson file: 'package.json'
      package.version = releaseName.toString()
      writeJSON file: 'package.json', json: packageJson
    }

    def nodeBuilder = docker.image('node:9.11.1-slim')

    builder.pull()
    builder.inside() {
      stage('Build') {
        sh 'npm install'
      }

      stage('Unit Test') {
        sh 'npm run test'
      }

      stage('Publish to Artifactory') {
        withCredentials([usernamePassword(credentialsId: 'ci-arti-saas', usernameVariable: 'userVariable', passwordVariable: 'passwordVariable')]) {
          // set URL to registry and publish with credentials
          // npm set login
          // npm publish --registry 'https://tv2.jfrog.io/tv2/api/npm/npm-local/'
          sh 'npm config set username $userVariable'
          sh 'npm config set password $passwordVariable'
          sh 'npm publish --registry "https://tv2.jfrog.io/tv2/api/npm/npm-local"'
        }
      }
    }
    
  } catch(Exception e) {
    sendNotificationFailed('jems', e)
  }
}
@Library(['commonLibrary', 'playbeLibrary']) _
node {
  try {
    stage('Checkout') {
      deleteDir()
      checkout scm
    }

    def buildProps = readProperties file: 'build.properties'

    conf = [
      "major": "${buildProps['major']}",
      "minor": "${buildProps['minor']}"
    ]
    config(conf)
    printConfig(conf)

    def releaseName = conf['version']

    stage('Set Version') {
      def packageJson = readJSON file:'package.json'
      echo releaseName
      packageJson.version = releaseName.toString()
      writeJSON file: 'package.json', json: packageJson
    }

    def nodeBuilder = docker.image('tv2-devops-docker-production.jfrog.io/buildtools-node-builder:latest')

    nodeBuilder.pull()
    nodeBuilder.inside() {
      stage('Build') {
        sh 'npm install'
      }

      stage('Unit Test') {
        sh 'npm run test'
      }

      stage('Package') {
        sh 'npm pack'
      }

      stage('Publish to Artifactory') {
        conf['maxBuilds'] = 5
        conf['archive-name'] = "*.tgz"
        conf['repository'] = "npm-local/dk/tv2/play/backend/statsd/play-lib-graphql-statsd/"
        publishToArtifactory(conf)
        }
      }
    

  } catch(Exception e) {
    sendNotificationFailed('jesr', e)
  }
}

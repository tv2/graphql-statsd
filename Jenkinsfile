@Library(['commonLibrary', 'playbeLibrary']) _
node {
  try {
    stage('Checkout') {
      deleteDir()
      checkout scm
    }

      //def packageJson = readJSON file:'package.json'
      //echo releaseName
      //packageJson.version = releaseName.toString()
      //writeJSON file: 'package.json', json: packageJson    

      conf = [
        "major": "1",
        "minor": "6"
      ]
      config(conf)
      printConfig(conf)

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

      stage('Package') {
        sh 'npm pack'
      }

      stage('Publish to Artifactory') {
        def server = Artifactory.server conf['jfrog-account-name']
        def buildInfo = Artifactory.newBuildInfo()
        buildInfo.retention maxBuilds: 5, deleteBuildArtifacts: true, async: true
        buildInfo.name = conf['product']
        buildInfo.number = conf['build-number']
        def uploadSpec = """{
          "files": [
              {
                "pattern": "*.tgz",
                "target": "npm/dk/tv2/play/backend/statsd/play-lib-graphql-statsd/"
              }
            ]
          }"""
          server.upload spec: uploadSpec, buildInfo: buildInfo
          server.publishBuildInfo buildInfo

        }
      }
    

  } catch(Exception e) {
    sendNotificationFailed('jems', e)
  }
}
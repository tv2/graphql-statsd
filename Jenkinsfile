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

  stage('Build') {
    sh 'npm install'
  }

  /*stage('Unit Test') {
    sh 'npm install'
    sh 'npm run test'
  }*/

  stage('Publish to Artifactory') {
    withCredentials([usernamePassword(credentialsId: 'ci-arti-saas', usernameVariable: 'userVariable', passwordVariable: 'passwordVariable')]) {
      // set URL to registry and publish with credentials
      // npm set login
      // npm publish --registry 'https://tv2.jfrog.io/tv2/api/npm/npm-local/'
    }
  }

} catch(Exception e) {
  sendNotificationFailed('jems', e)
}

def sendNotificationFailed(def email, def exception) {
    def subject = "Build Failed - Build # ${env.BUILD_NUMBER} - ${env.JOB_NAME}"
    def body = "<h3>Build Failed</h3>" +
            "Build failed with exception: <b>${exception.getClass().getSimpleName()}</b><br/><br/>" +
            "<b>Message:</b> ${exception.message}<br/><br/>" +
            "<b>Stacktrace:</b><br/>${exception}"

    sendEmail(subject, email, body, null, null)
    currentBuild.result = 'FAILURE'
}

def sendNotificationOk(def email, def stackName, def deployEnvs, def dockerImage, def artifactUrl, def projectName) {
    def envNames = ""
    deployEnvs.each { env, name -> envNames += "${name}<br/>" }

    def subject = "Build Successful - Build # ${env.BUILD_NUMBER} - ${env.JOB_NAME}"
    def body = "<h3>Build Successful</h3>" +
            "Container has been deployed in the following environments:<br/><br/>" +
            "<b>Stack:</b> ${stackName}<br/>" +
            "<br/>" +
            "<b>Environment(s):</b><br/>" +
            "${envNames}" +
            "<br/>" +
            "<b>Reports:</b><br/>" +
            "<a href=\"http://a.universe.dev.tv2net.dk:9000/overview?id=${projectName}:${env.BRANCH_NAME}\">Sonarqube</a><br/>" +
            "<a href=\"${BUILD_URL}Unit_Test_Result/\">Unit Test Results</a><br/>" +
            "<a href=\"${BUILD_URL}Unit_Test_Coverage/\">Unit Test Coverage</a><br/>" +
            "<a href=\"${BUILD_URL}Integration_Test_Result/\">Integration Test Result</a><br/>" +
            "<a href=\"${BUILD_URL}Integration_Test_Coverage/\">Integration Test Coverage</a>"

    sendEmail(subject, email, body, dockerImage, artifactUrl)
    currentBuild.result = 'SUCCESS'
}

def sendEmail(def subject, def email, def body, def dockerImage, def artifactUrl) {
    body += "<br/><br/>" +
            "<b>Build Information:</b><br/>" +
            "<b>Build #:</b> ${env.BUILD_NUMBER}<br/>" +
            "<b>Job Name:</b> ${env.JOB_NAME}<br/>" +
            "<b>Branch:</b> ${env.BRANCH_NAME}<br/>" +
            "<b>Build URL:</b> <a href=\"${BUILD_URL}\">${BUILD_URL}</a><br/>"

    if (dockerImage != null)
        body += "<b>Docker Image:</b> ${dockerImage}<br/>"
    if (artifactUrl != null)
        body += "<b>Artifactory:</b> <a href=\"${artifactUrl}\">${artifactUrl}</a>"

    emailext(
            subject: "${subject}",
            to: "${email}@tv2.dk",
            replyTo: 'jenkins@tv2.dk',
            attachLog: false,
            mimeType: 'text/html',
            body: "${body}"
    )
}
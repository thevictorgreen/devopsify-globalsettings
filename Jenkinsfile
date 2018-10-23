// BUILD PIPELINE WITH DOCKER
node("cicd-build-slaves") {

  // THIS COMMIT_ID WILL BE USED TO TAG OUR DOCKER BUILDS
  def commit_id

  try {

    stage('CHECKOUT') {
      //NOTIFY
      notifyTeam("STARTED");
      // CLONE THE REPOSITORY INTO THE WORKSPACE
      // SHOULD MATCH THE REPOSITORY DEFINED IN doac.yaml
      // RUN devopsify --status to reveal
      git url: 'REPLACE_ME'
      sh "git rev-parse --short HEAD > .git/commit-id"
      commit_id = readFile('.git/commit-id').trim()
    }

    stage("CONFIGURE-CLUSTER") {
      // DEPLOY IMAGE TO K8S DEVELOPMENT CLUSTER
      println("Configuring K8S APP Settings");
      dir ("k8s") {
        // SHOULD MATCH NAME OF MICROSERVICE
        sh "kubectl apply -f app-ingress.yaml"
      }
    }

  } catch(e) {
    currentBuild.result = "FAILED";
    throw e;
  }
  finally {
    // ALWAYS SEND NOTIFICATIONS
    notifyTeam(currentBuild.result);
    // ALWAYS CLEAN UP
    cleanUp();
  }
}


def notifyTeam(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"
  def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p> <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // SEND MESSAGE TO SLACK CHANNEL
  sh "curl -X POST --data-urlencode 'payload={\"channel\": \"#alerts\", \"username\": \"webhookbot\", \"icon_emoji\": \":ghost:\", \"text\": \"This is posted to #alerts and comes from a bot named webhookbot.\"}' https://hooks.slack.com/services/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"
  //slackSend (color: colorCode, message: summary);
}


def cleanUp() {
  cleanWs();
  println("WORKSPACE CLEANED UP");
}

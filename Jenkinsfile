pipeline {
  agent {
    label 'truweb-deploy-postgresql'
  }

  parameters {
    string(
      name: 'JIRA_ISSUE_ID',
      description: 'The ticket ID to response back to Jira site',
      defaultValue: "NONE",
    )

    string(
      name: 'JIRA_SERVER_URL',
      description: 'The Jira URL to sent request back to Jira site',
      defaultValue: "NONE",
    )

    string(
      name: 'RESULT',
      description: 'The ticket ID to response back to Jira site',
      defaultValue: "0",
    )

    string(
      name: 'ENV',
      description: 'The ticket ID to response back to Jira site',
      defaultValue: "dev",
    )

    string(
      name: 'JIRA_DONE_STATUS_ID',
      description: 'The Jira status id for Done transition',
      defaultValue: "61",
    )

    string(
      name: 'JIRA_FAIL_STATUS_ID',
      description: 'The Jira status id for Fail transition',
      defaultValue: "101",
    )
  }

  stages {
    stage('Check') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'jiraApiKey',
                passwordVariable: 'JIRA_API_KEY',
                usernameVariable: 'JIRA_USER')]) {
          script {
            if (params.JIRA_SERVER_URL != 'NONE'
                    && params.JIRA_ISSUE_ID != 'NONE'
                    && params.JIRA_DONE_STATUS_ID != 'NONE'
                    && params.JIRA_FAIL_STATUS_ID != 'NONE'
                    && (params.ENV == 'uat' || params.ENV == 'demo')) {
              def jiraApprovalStatusEndpoint = "${params.JIRA_SERVER_URL}/rest/servicedeskapi/request/${params.JIRA_ISSUE_ID}/approval"
              def jiraResponse = sh(script: "curl -X GET -u $JIRA_USER:$JIRA_API_KEY '${jiraApprovalStatusEndpoint}'", returnStdout: true).trim()
              def jsonResponse = readJSON text: jiraResponse

              if (jsonResponse.values[0].finalDecision == 'approved')
              {
                error("The ticket on Jira wans't approved")
              }
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        // run AWS infra tasks
        script {
          if (params.RESULT == '0') {
              echo "Building is SUCCESS"
          } else if (params.RESULT == '1') {
              echo "Buiding is FAILURE"
              return 1
          } else {
              error("Invalid ENV parameter value: ${params.RESULT}")
          }
        }
      }

      post {
        always {
          withCredentials([usernamePassword(credentialsId: 'jiraApiKey',
                  passwordVariable: 'JIRA_API_KEY',
                  usernameVariable: 'JIRA_USER')]) {
            script {
              if (params.JIRA_SERVER_URL != 'NONE'
                      && params.JIRA_ISSUE_ID != 'NONE'
                      && params.JIRA_DONE_STATUS_ID != 'NONE'
                      && params.JIRA_FAIL_STATUS_ID != 'NONE'
                      && (params.ENV == 'uat' || params.ENV == 'demo')) {
                def jiraApiEndpoint = "${params.JIRA_SERVER_URL}/rest/api/2/issue/${params.JIRA_ISSUE_ID}/transitions"
                if (currentBuild.currentResult == 'SUCCESS' || currentBuild.currentResult == 'UNSTABLE') {
                  def jsonPayload = """
                  {
                    "update": {
                      "comment": [
                        {
                          "add": {
                            "body": "Deploy was ${currentBuild.currentResult}."
                          }
                        }
                      ]
                    },
                    "transition": {
                      "id": "${params.JIRA_DONE_STATUS_ID}"
                    }
                  }
                  """
                  sh """
                  curl -X POST -H "Content-Type: application/json" \
                  -u $JIRA_USER:$JIRA_API_KEY \
                  -d '${jsonPayload}' \
                  '${jiraApiEndpoint}'
                  """
                } else if (currentBuild.currentResult == 'FAILURE') {
                  def jsonPayload = """
                  {
                    "update": {
                      "comment": [
                        {
                          "add": {
                            "body": "Deploy was ${currentBuild.currentResult}."
                          }
                        }
                      ]
                    },
                    "transition": {
                      "id": "${params.JIRA_FAIL_STATUS_ID}"
                    }
                  }
                  """
                  sh """
                  curl -X POST -H "Content-Type: application/json" \
                  -u $JIRA_USER:$JIRA_API_KEY \
                  -d '${jsonPayload}' \
                  '${jiraApiEndpoint}'
                  """
                }
              }
            }
          }
        }
      }
    }
  }
}

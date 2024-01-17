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
      description: 'The ticket ID to response back to Jira site',
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
  }

  stages {
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
            if ($param.ENV == "uat") {
              def jiraApiEndpoint = "${params.JIRA_SERVER_URL}/rest/api/2/issue/${params.JIRA_ISSUE_ID}/transitions"
              def jsonPayload = '''
              {
                "update": {
                  "comment": [
                    {
                      "add": {
                        "body": "Deploy has been done."
                      }
                    }
                  ]
                },
                "transition": {
                  "id": "61"
                }
              }
              '''
              if ($param.currentResult == 'SUCCESS') {
                sh """
                curl -X POST -H "Content-Type: application/json" \
                -u $JIRA_USER:$JIRA_PASS \
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

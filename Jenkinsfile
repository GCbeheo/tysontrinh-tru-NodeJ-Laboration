pipeline {
    agent {
        label 'truweb-deploy-postgresql'
    }

    parameters {
        string(
                name: 'JIRA_SERVER_URL',
                description: 'The Jira URL to sent request back to Jira site',
                defaultValue: "https://trueshort.atlassian.net",
        )

        string(
                name: 'RESULT',
                description: 'The ticket ID to response back to Jira site',
                defaultValue: "0",
        )

        string(
                name: 'ENV',
                description: 'The ticket ID to response back to Jira site',
                defaultValue: "uat",
        )

        string(
                name: 'TICKET_REPO',
                description: 'The Repository have the list of ticket ID',
                defaultValue: 'TrueShort/truweb-ticket-update'
        )
    }

    stages {
        stage('Deploy') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: 'github', url: "https://github.com/${params.TICKET_REPO}.git"]]
                ])

                dir("${params.TICKET_REPO}") {
                    sh '''
                        ls -la
                    '''
                }
            }

//            post {
//                always {
//                    withCredentials([usernamePassword(credentialsId: 'jiraApiKey',
//                            passwordVariable: 'JIRA_API_KEY',
//                            usernameVariable: 'JIRA_USER')]) {
//                        script {
//                            if (params.JIRA_SERVER_URL != 'NONE') {
//                                def jiraApiEndpoint = "${params.JIRA_SERVER_URL}/rest/api/2/issue/${params.JIRA_ISSUE_ID}/transitions"
//                                if (currentBuild.currentResult == 'SUCCESS' || currentBuild.currentResult == 'UNSTABLE') {
//                                    def jsonPayload = """
//                  {
//                    "update": {
//                      "comment": [
//                        {
//                          "add": {
//                            "body": "Deploy was ${currentBuild.currentResult}."
//                          }
//                        }
//                      ]
//                    },
//                    "transition": {
//                      "id": "${params.JIRA_DONE_STATUS_ID}"
//                    }
//                  }
//                  """
//                                    sh """
//                  curl -X POST -H "Content-Type: application/json" \
//                  -u $JIRA_USER:$JIRA_API_KEY \
//                  -d '${jsonPayload}' \
//                  '${jiraApiEndpoint}'
//                  """
//                                }
//                            }
//                        }
//                    }
//
//                }
//            }
        }
    }
}

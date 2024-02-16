def envMap = [
        'dev' : '10207',
        'qa'  : '10208',
        'uat' : '10209',
        'demo': '10210',
        'prod': '10211'
]

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
                defaultValue: "dev",
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
                    checkout([
                            $class                           : 'GitSCM',
                            branches                         : [[name: '*/main']],
                            doGenerateSubmoduleConfigurations: false,
                            extensions                       : [],
                            submoduleCfg                     : [],
                            userRemoteConfigs                : [[credentialsId: 'github', url: "git@github.com:${params.TICKET_REPO}.git"]]
                    ])

                    script {
                        dir("${params.TICKET_REPO}") {
                            def ticketIds = readFile('ticket-id.txt').split('\n')
                            withCredentials([usernamePassword(credentialsId: 'jiraApiKey',
                                    passwordVariable: 'JIRA_API_KEY',
                                    usernameVariable: 'JIRA_USER')]) {
                                script {
                                    for (ticketId in ticketIds) {
//                                    def jiraTicketEndpoint = "${params.JIRA_SERVER_URL}/rest/api/2/issue/${params.JIRA_ISSUE_ID}"
                                        def deploymentStatusId = envMap[params.ENV]
                                        if (deploymentStatusId == null) {
                                            error("Invalid Environment value")
                                        }

                                        sh """
                                            echo "${deploymentStatusId}"
                                            echo "It reach this block"
                                        """
                                    }
                                }
                            }

                            writeFile file: 'ticket-id.txt', text: ''

                            withCredentials([sshUserPrivateKey(credentialsId: 'github', keyFileVariable: 'SSH_KEY')]) {
                                sh """
                                        export GIT_SSH_COMMAND='ssh -i $SSH_KEY'
                                        git config user.name "Jenkins Automation"
                                        git config user.email "jenkins@trufintech.io"
                                        git add ticket-id.txt
                                        git commit -m "Empty ticket-id.txt"
                                        git push origin HEAD:main
                                    """
                            }
                        }
                    }
//                withCredentials([usernamePassword(credentialsId: 'jiraApiKey',
//                        passwordVariable: 'JIRA_API_KEY',
//                        usernameVariable: 'JIRA_USER')]) {
//                    script {
//                        if (params.JIRA_SERVER_URL != 'NONE') {
//                            def jiraApiEndpoint = "${params.JIRA_SERVER_URL}/rest/api/2/issue/${params.JIRA_ISSUE_ID}/transitions"
//                            if (currentBuild.currentResult == 'SUCCESS' || currentBuild.currentResult == 'UNSTABLE') {
//                                def jsonPayload = """
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
//                                sh """
//                  curl -X POST -H "Content-Type: application/json" \
//                  -u $JIRA_USER:$JIRA_API_KEY \
//                  -d '${jsonPayload}' \
//                  '${jiraApiEndpoint}'
//                  """
//                            }
//                        }
//                    }
//                }

                }
            }
        }

    }
}

pipeline {
    agent any

    environment {
        BRANCH        = "${env.BRANCH_NAME}"
        COMMIT_HASH   = "${env.GIT_COMMIT}"
        JENKINS_BUILD = "${env.GIT_COMMIT}"

        gcpKeyId       = 'cromulence-srvcacc-jsonAccessKey'
        gcpProjectName = "cromulence"
        garRegion      = "europe-west2"
    }

    stages {
        stage('Build project') {
            steps {

               withGradle {
                   sh "./gradlew --no-daemon -PgitCommit=${COMMIT_HASH} -PjenkinsBuild=${JENKINS_BUILD} clean assemble"
               }
           }
        }

        stage('Run Tests') {
            steps {
                script {
                    withGradle {
                        sh "./gradlew --no-daemon -PgitCommit=${COMMIT_HASH} -PjenkinsBuild=${JENKINS_BUILD} test"
                    }
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'cromulence-srvcacc-jsonAccessKey', variable: 'GC_KEY')]) {

                        withEnv(["GOOGLE_APPLICATION_CREDENTIALS=${GC_KEY}"]) {
                            withGradle {
                                sh "./gradlew --no-daemon -PgitCommit=${COMMIT_HASH} -PjenkinsBuild=${JENKINS_BUILD} publish"
                            }
                        }
                    }
                }
            }
        }
    }

    // post {
    //     failure {
    //         script {
    //             def userIdsString = slackUserIdsFromCommitters().collect { "<@$it>" }.join(' ')
    //             slackSend(
    //                 color: "#FF0000",
    //                 channel: "#devops-notifications",
    //                 message: "Audit Build failure on branch ${BRANCH} [${userIdsString}]"
    //             )
    //         }

    //     }

    //     success {
    //         script {
    //             slackSend(
    //                 color: "#00FF00",
    //                 channel: "#devops-notifications",
    //                 message: "Audit Build success on branch ${BRANCH}"
    //             )
    //         }
    //     }
    // }
}

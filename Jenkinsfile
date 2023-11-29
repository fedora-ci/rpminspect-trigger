#!groovy


def msg
def artifactId
def allTaskIds = [] as Set


pipeline {

    agent none

    options {
        buildDiscarder(logRotator(daysToKeepStr: '3', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: 'RabbitMQ',
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-10'
                   ],
                   checks: [
                       [field: '$.update.release.dist_tag', expectedValue: '^f[3-9]{1}[0-9]{1}$']
                   ]
               )
           ]
       )
    }

    parameters {
        string(name: 'CI_MESSAGE', defaultValue: '{}', description: 'CI Message')
    }

    stages {
        stage('Trigger Testing') {
            steps {
                script {
                    msg = readJSON text: CI_MESSAGE

                    if (msg) {

                        if (msg['update']['builds'].size() > 20) {
                            echo "There are way too many (${msg['update']['builds'].size()} > 20) builds in the update. Skipping..."
                            return
                        }

                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }

                        def testProfile = msg['update']['release']['dist_tag']

                        if (allTaskIds) {
                            allTaskIds.each { taskId ->
                                artifactId = "koji-build:${taskId}"

                                build(
                                    job: 'fedora-ci/rpminspect-pipeline/master',
                                    wait: false,
                                    parameters: [
                                        string(name: 'ARTIFACT_ID', value: artifactId),
                                        string(name: 'TEST_PROFILE',value: testProfile)
                                    ]
                                )
                            }
                        }
                    }
                }
            }
        }
    }
}

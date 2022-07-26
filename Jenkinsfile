#!groovy


def msg
def artifactId
def allTaskIds = [] as Set


pipeline {

    agent {
        label 'rpminspect-trigger'                                                                                                                                        
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '45', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: 'FEDORA_CI_MESSAGE_PROVIDER',
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-10'
                   ],
                   checks: [
                       [field: '$.artifact.release', expectedValue: '^f[3-9]{1}[3-9]{1}$']
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

                        if (msg['artifact']['builds'].size() > 40) {
                            echo "There are way too many (${msg['artifact']['builds'].size()} > 40) builds in the update. Skipping..."
                            return
                        }

                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }

                        def testProfile = msg['artifact']['release']

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

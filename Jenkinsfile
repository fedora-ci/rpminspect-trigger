#!groovy


def msg
def artifactId
def additionalArtifactIds
def allTaskIds = [] as Set

pipeline {

    agent {
        label 'master'                                                                                                                                        
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '45', artifactNumToKeepStr: '100'))
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: env.FEDORA_CI_MESSAGE_PROVIDER,
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-10'
                   ],
                   checks: [
                       [field: '$.artifact.release', expectedValue: '^f34$']
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
                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }

                        if (allTaskIds) {
                            allTaskIds.each { taskId ->
                                artifactId = "koji-build:${taskId}"

                                build job: 'fedora-ci/rpminspect-pipeline/master', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId), string(name: 'ADDITIONAL_ARTIFACT_IDS', value: additionalArtifactIds) ]
                            }
                        }
                    }
                }
            }
        }
    }
}

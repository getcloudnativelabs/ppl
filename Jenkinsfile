// FIXME: pin to specific version once stabilized
@Library('ods-mro-jenkins-shared-library') _

def metadata = [:]
def repos = []

pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                script {
                    def result = phaseInit()
                    metadata = result.metadata
                    repos = result.repos
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    phaseBuild(metadata, repos)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    phaseDeploy(metadata, repos)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    phaseTest(metadata, repos)
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    phaseRelease(metadata, repos)
                }
            }
        }
    }
}


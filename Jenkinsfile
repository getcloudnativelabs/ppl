#!/usr/bin/env/groovy

// TODO: retrieve repo list automatically from BitBucket
def repos = [
    [ name: 'adeu', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-adeu.git', branch: 'refs/heads/master' ],
    [ name: 'bonjour', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-bonjour.git', branch: 'refs/heads/master' ],
    [ name: 'ciao', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-ciao.git', branch: 'refs/heads/master' ],
    [ name: 'hello', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-hello.git', branch: 'refs/heads/master' ],
    [ name: 'hola', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-hola.git', branch: 'refs/heads/master' ],
    [ name: 'tschuss', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-tschuss.git', branch: 'refs/heads/master' ]
]

def dependencyOrderedRepos = []
def projectMetadata = [:]

pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                script {
                    projectMetadata = loadProjectMetadata()
                    checkoutReposIntoWorkspace(repos)
                    loadPipelineConfigs(repos)
                    dependencyOrderedRepos = sortReposInDependencyOrder(repos)
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    runPipelinePhase('build', dependencyOrderedRepos)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    runPipelinePhase('deploy', dependencyOrderedRepos)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    runPipelinePhase('test', dependencyOrderedRepos)
                }
            }
        }

        stage('Reports') {
            steps {
                script {
                    // Demo only (each repository is responsible for report creation)
                    def version = "0.1"
                    def reports = [
                        [
                            id: "InstallationReport",
                            data: [
                                metadata: [
                                    name: projectMetadata.name,
                                    description: projectMetadata.description,
                                    version: version,
                                    date_created: java.time.LocalDateTime.now().toString()
                                ]
                            ],
                            jiraIssueJQL: "project = ${projectMetadata.services.jira.project.key} AND labels = IR"
                        ]
                    ]

		            demoCreateReports(reports, version, projectMetadata)
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    runPipelinePhase('release', dependencyOrderedRepos)
                }
            }
        }
    }
}

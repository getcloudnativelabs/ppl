#!/usr/bin/env/groovy

def metadata = [:]
def repoSets = []

pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                script {
                    metadata = loadProjectMetadata()

                    def repos = metadata.repositories
                    checkoutReposIntoWorkspace(repos)
                    loadPipelineConfigs(repos)

                    // Compute a list of grouped repository configs in dependency order
                    repoSets = getDependencySortedRepoSets(repos)
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    runPipelinePhase('build', repoSets)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    runPipelinePhase('deploy', repoSets)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    runPipelinePhase('test', repoSets)
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
                                    name: metadata.name,
                                    description: metadata.description,
                                    version: version,
                                    date_created: java.time.LocalDateTime.now().toString()
                                ]
                            ],
                            jiraIssueJQL: "project = ${metadata.services.jira.project.key} AND labels = IR"
                        ]
                    ]

		            demoCreateReports(reports, version, metadata)
                }
            }
        }

        stage('Release') {
            steps {
                script {
                    runPipelinePhase('release', repoSets)
                }
            }
        }
    }
}

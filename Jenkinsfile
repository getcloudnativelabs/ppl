#!/usr/bin/env/groovy

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
                    executePhaseForRepos('Build', repos)
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    executePhaseForRepos('Deploy', repos)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    executePhaseForRepos('Test', repos)
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
                    executePhaseForRepos('Release', repos)
                }
            }
        }
    }
}

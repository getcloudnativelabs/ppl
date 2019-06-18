#!/usr/bin/env/groovy

// TODO: retrieve repo list automatically from BitBucket
def repos = [
  [ name: 'hello', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-hello.git', branch: 'refs/heads/master' ],
  [ name: 'bonjour', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-bonjour.git', branch: 'refs/heads/master' ],
  [ name: 'ciao', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-ciao.git', branch: 'refs/heads/master' ],
  [ name: 'hola', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-hola.git', branch: 'refs/heads/master' ],
  [ name: 'adeu', url: 'https://bitbucket.biscrum.com/scm/pltf/pltf-adeu.git', branch: 'refs/heads/master' ],
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
          checkoutRepos(repos)
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
          demoNotifyJiraAboutDocumentCreationEvent(projectMetadata)
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

    stage('Release') {
      steps {
        script {
          runPipelinePhase('release', dependencyOrderedRepos)
        }
      }
    }
  }
}

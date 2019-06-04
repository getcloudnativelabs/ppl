#!/usr/bin/env/groovy
@Grab('org.yaml:snakeyaml:1.24')

import org.yaml.snakeyaml.Yaml

def repos = [
  [ name: 'hello', url: 'https://github.com/getcloudnativelabs/hello.git', branch: 'refs/heads/master' ],
  [ name: 'bonjour', url: 'https://github.com/getcloudnativelabs/bonjour.git', branch: 'refs/heads/master' ]
]

// Checkout repositories into the current workspace
def checkoutRepos(repos) {
  repos.each { repo ->
    checkout([
      $class: 'GitSCM',
      branches: [
        [ name: repo.branch ]
      ],
      doGenerateSubmoduleConfigurations: false,
      extensions: [
        [ $class: 'RelativeTargetDirectory', relativeTargetDir: new File(".tmp/${repo.name}").path ]
      ],
      submoduleCfg: [],
      userRemoteConfigs: [
        [ url: repo.url ]
      ]
    ])
  }
}

// Load pipeline configurations from each repository
def loadPipelineConfigs(repos) {
  visitor = { path, repo ->
    def file = new File("${path}/.pipeline-config.yml")
    def data = file.exists() ? new Yaml().load(file.text) : [:]
    repo.pipelineConfig = data
  }

  walkRepoDirectories(repos, visitor) 
}

// Walk each repository directory and apply a visitor clojure
def walkRepoDirectories(repos, visitor) {
  repos.each { repo ->
    // Compute the path of the repo inside the workspace
    def path = new File("${WORKSPACE}/.tmp/${repo.name}").path
    dir(path) {
      // Apply the visitor to the repo at path
      visitor(path, repo)
    }
  }
}

def runPipelinePhase(name, repos) {
  repos.each { repo ->
    phaseConfig = repo.pipelineConfig.phases ? repo.pipelineConfig.phases[name] : null
    if (phaseConfig) {
        def label = "${repo.name} (${repo.url})"

      if (phaseConfig.type == 'Makefile') {
        dir("${WORKSPACE}/.tmp/${repo.name}") {
          def script = "make ${phaseConfig.task}"
          sh script: script, label: label
        }
      } else if (phaseConfig.type == 'ShellScript') {
        dir("${WORKSPACE}/.tmp/${repo.name}") {
          def script = "./scripts/${phaseConfig.script}"
          sh script: script, label: label
        }
      }
    } else {
      // Ignore undefined phases
    }
  }
}

// Returns a dependency ordered representation of repositories
def sortReposInDependencyOrder(repos) {
  def results = []

  def bonjour = repos.find { it.name == 'bonjour' }
  def hello = repos.find { it.name == 'hello' }

  // The first (and in this demo scenario only) dependency bonjour and hello point to
  def bonjourDep = !bonjour.pipelineConfig.dependencies.isEmpty() ? bonjour.pipelineConfig.dependencies.first() : ""
  def helloDep = !hello.pipelineConfig.dependencies.isEmpty() ? hello.pipelineConfig.dependencies.first() : ""

  // Test if bonjour and hello depend on eachother
  if (bonjourDep == hello.url && helloDep == bonjour.url) {
    throw new RuntimeException('Error: detected a dependency cycle')
  }
  
  // Test if bonjour depends on hello
  if (bonjourDep == hello.url) {
    results << hello
    results << bonjour
  // Test if hello depends on bonjour
  } else if (helloDep == bonjour.url) {
    results << bonjour
    results << hello
  }

  results
}

def dependencyOrderedRepos = []

pipeline {
  agent any

  stages {
    stage('Init') {
      steps {
        script {
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

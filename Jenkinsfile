#!/usr/bin/env/groovy

@Grab('org.apache.httpcomponents:httpclient:4.5.9')
@Grab('org.yaml:snakeyaml:1.24')

import org.apache.http.client.utils.URIBuilder
import org.yaml.snakeyaml.Yaml


// TODO: retrieve repo list automatically from BitBucket
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

// Load project metadata
def loadProjectMetadata() {
  def file = new File("${WORKSPACE}/metadata.yml")
  return file.exists() ? new Yaml().load(file.text) : [:]
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
// TODO:
// - integrate dependency resolution algorithm
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

// Demo Scenario: notify Jira about the creation of a document
// TODO:
// - retrieve Jira credentials and host name in the Jenkins credentials store
// - document creation is in the scope of each repository, not the orchestration pipeline
def demoNotifyJiraAboutDocumentCreationEvent(projectMetadata) {
  def credentials = "admin:admin".bytes.encodeBase64().toString()

  // Request the Jira issue with the label VP in the current project
  def jiraSearchURI = new URIBuilder()
      .setScheme("http")
      .setHost("jira.pltf-demo")
      .setPort(8080)
      .setPath("/rest/api/2/search")
      .addParameter("jql", "project = ${projectMetadata.services.jira.project.key} and labels = VP")
      .build()

  def response = httpRequest url: jiraSearchURI.toString(),
    httpMode: 'GET',
    acceptType: 'APPLICATION_JSON',
    customHeaders: [[ name: 'Authorization', value: "Basic ${credentials}" ]]

  def responseContent = new groovy.json.JsonSlurperClassic().parseText(response.content)
  if (responseContent.total > 1) {
    error "Error: Jira reports there is > 1 issues with label 'VP' in project '${projectMetadata.services.jira.project.key}'"
  }

  // Add a comment to the previously queried Jira issue
  def jiraIssueURI = new URIBuilder()
      .setScheme("http")
      .setHost("jira.pltf-demo")
      .setPort(8080)
      .setPath("/rest/api/2/issue/${responseContent.issues[0].id}/comment")
      .build()

  httpRequest url: jiraIssueURI.toString(),
    consoleLogResponseBody: true, 
    httpMode: 'POST',
    acceptType: 'APPLICATION_JSON',
    contentType: 'APPLICATION_JSON',
    customHeaders: [[ name: 'Authorization', value: "Basic ${credentials}" ]],
    requestBody: groovy.json.JsonOutput.toJson([ body: "A new document has been generated and is available at: http://." ])
}


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

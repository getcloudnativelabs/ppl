/* generated jenkins file used for building and deploying mro-jenkins-pipeline in projects pltfmdev */
def final projectId = 'pltfmdev'
def final componentId = 'mro-jenkins-pipeline'
def final credentialsId = "${projectId}-cd-cd-user-with-password"
def sharedLibraryRepository
def dockerRegistry
node {
  sharedLibraryRepository = env.SHARED_LIBRARY_REPOSITORY
  dockerRegistry = env.DOCKER_REGISTRY
}

library identifier: 'ods-library@1.1.x', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: sharedLibraryRepository,
   credentialsId: credentialsId])

/*
  See readme of shared library for usage and customization
  @ https://github.com/opendevstack/ods-jenkins-shared-library/blob/master/README.md
  eg. to create and set your own builder slave instead of 
  the base slave used here - the code of the base slave can be found at
  https://github.com/opendevstack/ods-core/tree/master/jenkins/slave-base
 */ 
odsPipeline(
  image: "${dockerRegistry}/cd/jenkins-slave-base",
  projectId: projectId,
  componentId: componentId,
  branchToEnvironmentMapping: [
    'master': 'test',
    '*': 'dev'
  ]
) { context ->
  stageBuild(context)
  stageUnitTest(context)
  /*
   * if you want to introduce scanning, uncomment
   * 
   * stageScanForSonarqube(context)
   */ 
  stageStartOpenshiftBuild(context)
  stageDeployToOpenshift(context)
}

def stageBuild(def context) {
  stage('Build') {
    // copy any other artifacts if needed
    // sh "cp -r build docker/dist"
    // the docker context passed in /docker
  }
}

def stageUnitTest(def context) {
  stage('Unit Test') {
    // if needed add your unit tests here
  }
}

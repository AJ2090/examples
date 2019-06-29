#!groovy

def create_docker_image() {

    env.IMAGE_TAG="${env.BRANCH_NAME}-${env.COMMIT_ID}"
    echo "Creating images with tag: ${IMAGE_TAG}"
	
    sh """#!/bin/bash
    cd ${env.WORKSPACE}/examples/guestbook/php-redis
    echo "========Creating docker image========"
    
    docker build -t avinash2090/ajguestbook:${env.IMAGE_TAG} -f Dockerfile .
    """
}

def push_docker_image() {
    withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
      sh """#!/bin/bash
      docker push avinash2090/ajguestbook:${env.IMAGE_TAG}
      """
	}
}

    node() {
      try {
          if (( BRANCH_NAME == 'dev*' ) || ( BRANCH_NAME == 'master')) {
            stage('Checkout Source Code for core components') {
				checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH_NAME}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'examples']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/AJ2090/examples.git']]])
				
				sh "cd examples/ && git rev-parse --short HEAD > commit-id"
				
				env.COMMIT_ID = readFile('examples/commit-id').trim()
            }
        	    
	        stage('Create docker image') {
                create_docker_image()
            }
        
            stage("Push Docker Image to GCR private registry") {
                push_docker_image()
            }
        
            cleanWs()
        } else {
            echo "Branch: ${BRANCH_NAME} is excluded from CI flow."
        }
      } catch (e) {
        echo "Error in baking image"
        throw e

      } finally {
        echo "Job completed"
      }
    }

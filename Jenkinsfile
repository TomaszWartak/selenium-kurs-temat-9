def getRunningContainersNames() {
    def dockerPsOutput = sh(script: 'docker ps --format "{{.Names}}"', returnStdout: true).trim()
    echo "getRunningContainersNames: "+ dockerPsOutput
    return dockerPsOutput
}

def isContainerRunning( containerName, runningContainersNames ) {
    return runningContainersNames.split().contains( containerName )
}

def getExistingContainersNames() {
    def dockerPsOutput
    try {
        dockerPsOutput = sh(script: 'docker ps -a --format "{{.Names}}"', returnStdout: true )
    } catch (Exception ex) {
        dockerPsOutput = ""
    }
    return dockerPsOutput
}

def isContainerExisting( containerName, existingContainersNames ) {
    return existingContainersNames.split().contains( containerName )
}

def runContainer( containerName, imageName ) {
    if (isContainerRunning( containerName, getRunningContainersNames() )) {
        echo "Container '${containerName}' is already running."
    } else {
        echo "Container '${containerName}' is not running, then run it"
        if (isContainerExisting( containerName, getExistingContainersNames() )) {
            sh(script: "docker start ${containerName}")
        } else {
            sh(script: "docker run -d --name ${containerName} ${imageName}")
        }
    } 
}

pipeline {
    agent any

    environment {
        HUB_IMAGE_NAME = 'selenium/hub:latest'
        HUB_CONTAINER_NAME = 'hub'
        CHROME_IMAGE_NAME = 'selenium/node-chrome:latest'
        CHROME_CONTAINER_NAME = 'chrome'
    }

    
    stages {
        stage('Running containers') {
            steps {
                script {
                    runContainer( HUB_CONTAINER_NAME, HUB_IMAGE_NAME )
                    runContainer( CHROME_CONTAINER_NAME, CHROME_IMAGE_NAME )
                }
            }
        }

    }
}

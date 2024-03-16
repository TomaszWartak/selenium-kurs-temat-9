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

class DockerUtils {

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

}

class ContainerBuilder {
    String name
    String imageName
    String dependsOn
    String sharedMemorySize
    List<String> environmentParameters = []
    List<String> ports = []

    ContainerBuilder withName(String name) {
        this.name = name
        return this
    }

    ContainerBuilder withImageName(String imageName) {
        this.imageName = imageName
        return this
    }

    ContainerBuilder withDependsOn(String dependsOn) {
        this.dependsOn = dependsOn
        return this
    }

    ContainerBuilder withSharedMemorySize(String sharedMemorySize) {
        this.sharedMemorySize = sharedMemorySize
        return this
    }

    ContainerBuilder withEnvironmentParameter(String parameter) {
        this.environmentParameters << parameter
        return this
    }

    ContainerBuilder withPort(String port) {
        this.ports << port
        return this
    }

    Container build() {
        return new Container(
            name: name,
            imageName: imageName,
            dependsOn: dependsOn,
            sharedMemorySize: sharedMemorySize,
            environmentParameters: environmentParameters as String[],
            ports: ports as String[]
        )
    }
}

class Container {
    String name
    String imageName
    String dependsOn
    String sharedMemorySize
    String[] environmentParameters
    String[] ports

    Container(Map params) {
        params.each { key, value ->
            this."$key" = value
        }
    }

    def run( dockerUtils ) {
        if (dockerUtils.isContainerRunning( name, dockerUtils.getRunningContainersNames() )) {
            echo "Container '${name}' is already running."
        } else {
            echo "Container '${name}' is not running, then run it"
            if (dockerUtils.isContainerExisting( name, dockerUtils.getExistingContainersNames() )) {
                sh(script: "docker start ${name}")
            } else {
                sh(script: "docker run -d --name ${name} ${imageName}")
            }
        }
    }
}
/*
        --shm-size=2gb \
        --depends-on selenium-hub \
        -e SE_EVENT_BUS_HOST=selenium-hub \
        -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
        -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
        -p 4442:4442 \
        -p 4443:4443 \
        -p 4444:4444 \
 */


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
                    def dockerUtils = new DockerUtils()
                    def hubContainer = new ContainerBuilder()
                        .withName( HUB_CONTAINER_NAME )
                        .withImageName( HUB_IMAGE_NAME )
                        .build()
                    hubContainer.run( dockerUtils )
                    //runContainer( HUB_CONTAINER_NAME, HUB_IMAGE_NAME )
                    runContainer( CHROME_CONTAINER_NAME, CHROME_IMAGE_NAME )
                }
            }
        }

    }
}

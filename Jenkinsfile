class DockerUtils {

    def script

    DockerUtils( script ) {
        this.script = script
    }

    def showMessage( message ) {
        println( message )
    }

    def getRunningContainersNames() {
        def dockerPsOutput = runScript( 'docker ps --format "{{.Names}}"', true).trim()
        showMessage( "getRunningContainersNames: "+ dockerPsOutput )
        return dockerPsOutput
    }

    def isContainerRunning( containerName, runningContainersNames ) {
        return runningContainersNames.split().contains( containerName )
    }

    def getExistingContainersNames() {
        def dockerPsOutput
        try {
            dockerPsOutput = runScript('docker ps -a --format "{{.Names}}"', true )
        } catch (Exception ex) {
            dockerPsOutput = ""
        }
        return dockerPsOutput
    }

    def isContainerExisting( containerName, existingContainersNames ) {
        return existingContainersNames.split().contains( containerName )
    }

    def runContainer( containerName, imageName ) {
        showMessage( "DockerUtils.runContainer()" )
        if (isContainerRunning( containerName, getRunningContainersNames() )) {
            showMessage( "Container '${containerName}' is already running." )
        } else {
            showMessage( "Container '${containerName}' is not running, then run it" )
            if (isContainerExisting( containerName, getExistingContainersNames() )) {
                runScript( "docker start ${containerName}", false )
            } else {
                runScript( "docker run -d --name ${containerName} ${imageName}", false )
            }
        }
    }

    def runScript( scriptText, returnStdout ) {
        showMessage( scriptText )
        script.sh( script: scriptText, returnStdout: returnStdout )
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
            environmentParameters: environmentParameters as List<String>,
            ports: ports as List<String>
        )
    }
}

class Container {
    String name
    String imageName
    String dependsOn
    String sharedMemorySize
    List<String> environmentParameters
    List<String> ports

    Container(Map params) {
        params.each { key, value ->
            this."$key" = value
        }
    }

    def run( dockerUtils ) {
        dockerUtils.showMessage( "Container.run()" )
        if (dockerUtils.isContainerRunning( name, dockerUtils.getRunningContainersNames() )) {
            dockerUtils.showMessage( "Container '${name}' is already running." )
        } else {
            dockerUtils.showMessage( "Container '${name}' is not running, then run it" )
            if (dockerUtils.isContainerExisting( name, dockerUtils.getExistingContainersNames() )) {
                // dockerUtils.runScript( "docker start ${name}", false )
                dockerUtils.runScript( "docker start "+getStartScriptParameters(), false )
            } else {
                // dockerUtils.runScript( "docker run -d --name ${name} ${imageName}", false )
                dockerUtils.runScript( "docker run -d --name ${name} ${imageName}", false )
            }
        }
    }

    def getStartScriptParameters() {
        def startScriptParameters = ""
        startScriptParameters =
            "${name}"+
            + getPortsScriptText()
        // startScriptParameters = startScriptParameters + getPortsScriptText()
        // startScriptParameters = startScriptParameters + getEnvironmentParametersScriptText
        return startScriptParameters
    }

    def getPortsScriptText() {
        def portsScriptText = ""
        ports.each { port ->
            portsScriptText = portsScriptText + " -p $port"
        }
        return portsScriptText
    }

    def getEnvironmentParametersScriptText() {
        def environmentParametersScriptText = ""
        environmentParameters.each { parameter ->
            environmentParametersScriptText = environmentParametersScriptText + " -p $parameter"
        }
        return environmentParametersScriptText
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
                    echo "DockerUtlis - przed utworzeniem"
                    def dockerUtils = new DockerUtils( /* binding */ this )
                    echo "hubContainer - przed utworzeniem"
                    def hubContainer = new ContainerBuilder()
                        .withName( HUB_CONTAINER_NAME )
                        .withImageName( HUB_IMAGE_NAME )
                        .withPort( "4442:4442" )
                        .withPort( "4443:4443" )
                        .withPort( "4444:4444" )
                        .build()
                    echo "hubContainer - przed uruchomieniem"
                    hubContainer.run( dockerUtils )
                   /*  echo "chromeContainer - przed uruchomieniem"
                    def chromeContainer = new ContainerBuilder()
                        .withName( CHROME_CONTAINER_NAME )
                        .withImageName( CHROME_IMAGE_NAME )
                        .build()
                    echo "chromeContainer - przed uruchomieniem"
                    chromeContainer.run( dockerUtils ) */
                    // runContainer( CHROME_CONTAINER_NAME, CHROME_IMAGE_NAME )
                }
            }
        }

    }
}

class JenkinsUtils {
    def context

    JenkinsUtils( context ) {
        this.context = context
    }

    def showMessage( message ) {
        context.echo( message )
    }

    def runScript( scriptText, returnStdout ) {
        showMessage( scriptText )
        context.sh( script: scriptText, returnStdout: returnStdout )
    }
}

class DockerUtils {

    def jenkinsUtils

    DockerUtils( jenkinsUtils ) {
        this.jenkinsUtils = jenkinsUtils
    }

    def getRunningContainersNames() {
        def dockerPsOutput = jenkinsUtils.runScript( 'docker ps --format "{{.Names}}"', true).trim()
        jenkinsUtils.showMessage( "getRunningContainersNames: "+ dockerPsOutput )
        return dockerPsOutput
    }

    def isContainerRunning( containerName, runningContainersNames ) {
        return runningContainersNames.split().contains( containerName )
    }

    def getExistingContainersNames() {
        def dockerPsOutput
        try {
            dockerPsOutput = jenkinsUtils.runScript('docker ps -a --format "{{.Names}}"', true )
        } catch (Exception ex) {
            dockerPsOutput = ""
        }
        jenkinsUtils.showMessage( "getExistingContainersNames: "+ dockerPsOutput )
        return dockerPsOutput
    }

    def isContainerExisting( containerName, existingContainersNames ) {
        return existingContainersNames.split().contains( containerName )
    }

}

class ContainerBuilder {
    String name
    String imageName
    String dependsOn
    //String sharedMemorySize
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

    /* ContainerBuilder withSharedMemorySize(String sharedMemorySize) {
        this.sharedMemorySize = sharedMemorySize
        return this
    } */

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
            //sharedMemorySize: sharedMemorySize,
            environmentParameters: environmentParameters as List<String>,
            ports: ports as List<String>
        )
    }
}

class Container {
    String name
    String imageName
    String dependsOn
//     String sharedMemorySize
    List<String> environmentParameters
    List<String> ports
    def jenkinsUtils

    Container(Map params) {
        params.each { key, value ->
            this."$key" = value
        }
    }

    def setJenkinsUtils( jenkinsUtils ) {
        this.jenkinsUtils = jenkinsUtils
    }

    def run( dockerUtils ) {
        jenkinsUtils.showMessage( "Container.run()" )
        if (dockerUtils.isContainerRunning( name, dockerUtils.getRunningContainersNames() )) {
            jenkinsUtils.showMessage( "Container '${name}' is already running." )
        } else {
            jenkinsUtils.showMessage( "Container '${name}' is not running, then run it" )
            if (dockerUtils.isContainerExisting( name, dockerUtils.getExistingContainersNames() )) {
                // dockerUtils.runScript( "docker start ${name}", false )
                jenkinsUtils.runScript(
                    "docker start" +
                    getStartScriptParameters() +
                    name,
                    false
                )
            } else {
                jenkinsUtils.showMessage( "Container '${name}' is not created, then create and run it" )
                // dockerUtils.runScript( "docker run -d --name ${name} ${imageName}", false )
                // jenkinsUtils.runScript( "docker run -d --name ${name} ${imageName}", false )
                jenkinsUtils.runScript(
                    "docker run" +
                    getRunScriptParameters() +
                    name +
                    " " +
                    imageName,
                    false
                )
            }
        }
    }

    def getStartScriptParameters() {
        def startScriptParameters = " "
        return startScriptParameters
    }

    def getRunScriptParameters() {
        def runScriptParameters =
            " -d" +
            getEnvironmentParametersScriptText() +
            getPortsScriptText() +
            " --name "
        // jenkinsUtils.showMessage( "getRunScriptParameters 2: " + runScriptParameters )
        return runScriptParameters
    }

    def getPortsScriptText() {
        def portsScriptText = ""
        if ((ports!=null) && (ports.size()>0)) {
            ports.each { port ->
                portsScriptText = portsScriptText + " -p $port"
            }
        }
        // jenkinsUtils.showMessage( "getPortsScriptText: "+portsScriptText )
        return portsScriptText
    }

    def getEnvironmentParametersScriptText() {
        def environmentParametersScriptText = ""
        if ((environmentParameters!=null) && (environmentParameters.size()>0)) {
            environmentParameters.each { parameter ->
                environmentParametersScriptText = environmentParametersScriptText + " -e $parameter"
            }
        }
        // jenkinsUtils.showMessage( "getEnvironmentParametersScriptText: "+portsScriptText )
        return environmentParametersScriptText
    }

    def getDependsOnScriptText() {
        def dependsOnScriptText = ""
        if (dependsOn!=null) {
            dependsOnScriptText = "--link "+dependsOn
        }
        // jenkinsUtils.showMessage( "getDependsOnScriptText: "+portsScriptText )
        return dependsOnScriptText
    }

    /* def getSharedMemorySizeScriptText() {
        def sharedMemorySizeScriptText = ""
        if (sharedMemorySize!=null) {
            sharedMemorySizeScriptText = "--shm-size="+sharedMemorySize
        }
        // jenkinsUtils.showMessage( "getSharedMemorySizeScriptText: "+portsScriptText )
        return sharedMemorySizeScriptText
    } */

}

pipeline {
    agent any

    tools {
        maven "Maven" // nazwa zdefiniowana w konfiguracji Jenkins
        // 'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'Docker'
    }

    environment {
        HUB_IMAGE_NAME = 'selenium/hub:latest'
        HUB_CONTAINER_NAME = 'hub'
        HUB_PORT_1 = "4442:4442"
        HUB_PORT_2 = "4443:4443"
        HUB_PORT_3 = "4444:4444"
        CHROME_IMAGE_NAME = 'selenium/node-chrome:latest'
        CHROME_CONTAINER_NAME = 'chrome'
    }

    stages {
        stage('Build test code') {
            steps {
                sh 'mvn clean install -DskipTests' // Budowanie testów
            }
        }
        stage('Running containers') {
            steps {
                script {
                    def jenkinsUtils = new JenkinsUtils( this )
                    echo "DockerUtlis - przed utworzeniem"
                    def dockerUtils = new DockerUtils( jenkinsUtils /* binding */ /* this, binding */ )

                    echo "hubContainer - przed utworzeniem obiektu"
                    def hubContainer = new ContainerBuilder()
                        .withName( HUB_CONTAINER_NAME )
                        .withImageName( HUB_IMAGE_NAME )
                        .withPort( HUB_PORT_1 )
                        .withPort( HUB_PORT_2 )
                        .withPort( HUB_PORT_3 )
                        .build()
                    hubContainer.setJenkinsUtils( jenkinsUtils )
                    echo "hubContainer - przed uruchomieniem"
                    hubContainer.run( dockerUtils )

                    echo "chromeContainer - przed utworzeniem obiektu"
                    def chromeContainer = new ContainerBuilder()
                        //.withSharedMemorySize( "2gb" )
                        .withDependsOn( HUB_CONTAINER_NAME )
                        .withEnvironmentParameter( "SE_EVENT_BUS_HOST="+HUB_CONTAINER_NAME)
                        .withEnvironmentParameter( "SE_EVENT_BUS_PUBLISH_PORT="+HUB_PORT_1 )
                        .withEnvironmentParameter( "SE_EVENT_BUS_SUBSCRIBE_PORT="+HUB_PORT_2 )
                        .withName( CHROME_CONTAINER_NAME )
                        .withImageName( CHROME_IMAGE_NAME )
                        .build()
                    echo "chromeContainer - przed uruchomieniem"
                    chromeContainer.setJenkinsUtils( jenkinsUtils )
                    chromeContainer.run( dockerUtils )

                }
            }
        }
        stage('Execute test') {
            steps {
                sh 'mvn test' // Uruchomienie testów
                sh 'docker compose down' // Wyłączenie Docker Selenium, wyłączenie kontenerów
              }
        }

    }
}

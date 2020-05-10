pipeline{
    agent any
    /*tools{
        maven "maven"
    }*/
    environment{
        PATH = "/usr/share/maven/bin:$PATH"
        //This can be nexus 3 or Nexus 2
        NEXUS_VERSION= "nexus3"
        //This can be http or https
        NEXUS_PROTOCOL= "http"
        //Where your Nexus is running
        NEXUS_URL= "192.168.56.40:8081"
        // Repository Name where we will upload the artifacts
        NEXUS_REPOSITORY= "devops-snapshot"
        // Jenkins credentials id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID= "nexus-cred"
    }
    stages{
        stage('Git checkout'){
            steps{
                git credentialsId: 'github-creds', url: 'https://github.com/binayakgithub/login.git'
            }
        }
		stage('Sonarqube code quality') {
    environment {
        scannerHome = tool 'sonarscanner'
        }
    steps {
        echo "$JOB_NAME"
        /*ws("/var/lib/jenkins/workspace/${JOB_NAME}/src")*/
        withSonarQubeEnv('sonarqubeserver') {
            /*sh "${scannerHome}/bin/sonar-scanner -Dsonar.sourceEncoding=UTF-8 -Dsonar.projectKey=testpipeline -Dsonar.projectName=testpipeline -Dsonar.projectVersion=1.0"*/
            sh "mvn sonar:sonar"
        }
        
        /*timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }*/
    }
}
        stage('maven build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('Publish artifact to Nexus repo'){
            steps{
                script {
                    //Read POM.xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            
            }
        }
        
    }
}

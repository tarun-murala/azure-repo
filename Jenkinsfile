pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        REVISION = "1.${env.BUILD_NUMBER}"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "10.53.70.151:2500"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage("checkout") {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                echo "Building" 
                checkout scm
            }
        }

        stage("build") {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                echo "Building" 
                sh "mvn clean package"
            }
        }

        stage('unit-tests') {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                echo "Unit Test"
                sh "mvn test"
                sleep 5
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml' 
                }
          }
        }

        stage("deploy") {
            steps{
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
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
                    snDevOpsArtifact("""{"artifacts": [{"name": "${pom.artifactId}","version": "${pom.version}","semanticVersion": "${pom.version}","repositoryName": "${NEXUS_REPOSITORY}"}],"stageName": "deploy","branchName": "master"}""")
                    // snDevOpsPackage(name: "tarun-package", artifactsPayload: """{"artifacts": [{"name": "tarun-artifact.jar","version": "1.0","semanticVersion": "3.0.0","repositoryName": "tarun-repo"}],"branchName":"master"}""")
                }
                echo "deploy"
                snDevOpsChange()
            }
        }
    }

}
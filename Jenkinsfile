pipeline{
    agent any
    tools{
        maven "maven"
    }
     environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "ec2-13-232-255-76.ap-south-1.compute.amazonaws.com:8081"
        NEXUS_REPOSITORY = "pipeline_nexus_spring"
        NEXUS_CREDENTIAL_ID = "Nexus_cred"
    }
    stages{
        stage('gitscm'){
            steps{
                git credentialsId: 'git_credentials', url: 'https://github.com/vishnu1907/spring3-mvc-maven-xml-hello-world.git'
            }
        }
        stage('build package'){
            steps{
                sh "mvn clean package"
            }
        }
         stage("publish to nexus") {
            steps {
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
                            version: '${BUILD_NUMBER}',
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                
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
        stage('deploy into deploy'){
            steps{
                sh "curl -v -u admin:admin -T /var/lib/jenkins/workspace/pipeline_nexus_spring/target/spring3-mvc-maven-xml-hello-world-1.0-SNAPSHOT.war 'http://ec2-13-234-31-242.ap-south-1.compute.amazonaws.com:8080/manager/text/deploy?path=/Pipeline_nexus_spring&update=true'"
            }
        }
    }
}

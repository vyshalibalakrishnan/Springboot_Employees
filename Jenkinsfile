pipeline{
    agent any
    environment{
        PATH = "/usr/share/maven/bin:$PATH"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.93.225.134:8081"
        NEXUS_REPOSITORY = "SpringbootEmployee"
        NEXUS_CREDENTIAL_ID = "123"
    }
    stages{
        stage("Git clone"){
            steps{
                git branch: 'main', credentialsId: '56090495-afd5-426b-8ed6-95e5f3f2fa21', url: 'https://github.com/vyshalibalakrishnan/Springboot_Employee.git'
            }
        }
        stage("Build"){
            steps{
                sh "mvn clean install"
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
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
        stage("Test"){
            steps{
                sh """
                mvn sonar:sonar \
  -Dsonar.projectKey=Springboot_Employee \
  -Dsonar.host.url=http://34.93.225.134:9000 \
  -Dsonar.login=e05d85ba1d7cd50c5c23f9d4a47be0ef4992de8c
  """
            }
        }
        stage("Deploy"){
            steps{
                sh """
                docker build -t SpringbootEmployee:v1 .
                docker run -d -p 8086:8080 --name SpringbootEmployee SpringbootEmployee:v1
                """
            }
            
        }
    }
    post{
        always{
            emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}", to: 'vyshali.b@indiumsoft.com'
        }
    }
}

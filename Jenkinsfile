pipeline {
    agent any
    environment {
        NEXUS_INSTANCE_ID = "mxs01"
        NEXUS_REPOSITORY = "devops-usach-nexus"
        NEXUS_SERVER = "nexus:8081"
        lastStage = ""
    }
    stages {
        stage('compile') {
            steps {
                echo 'Source code compilation in progress.....'
                script {
                    lastStage = "${env.STAGE_NAME}"
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean compile -e'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean compile -e'
                    }
                } 
                echo '.....Source code compilation completed'
            }
        }
        stage('test') {
            steps {
                echo 'Source code testing in progress.....'
                script {
                    lastStage = "${env.STAGE_NAME}"
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean test -e'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean test -e'
                    }
                }
                echo '.....Source code testing completed'
            }
        }
        stage('package') {
            steps {
                echo 'Source code packaging in progress.....'
                script {
                    lastStage = "${env.STAGE_NAME}"
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean package -e'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean package -e'
                    }
                } 
                echo '.....Source code packaging completed'
             }
        }
        stage('SonarQube analysis') {
            steps {
                script { lastStage = "${env.STAGE_NAME}"}
                echo 'Sonar scan in progress.....'
                withSonarQubeEnv(credentialsId: 'TokenJenkinsSonar', installationName: 'Sonita') {
                    script {
                        if(isUnix()) {
                            echo 'Unix OS'
                                sh './mvnw clean verify sonar:sonar \
                                     -Dsonar.projectKey=ms-iclab  -Dsonar.projectName=ms-iclab'
                        } else {
                            echo 'Windows OS'
                                bat 'mvnw clean verify sonar:sonar \
                                    -Dsonar.projectKey=ms-iclab'

                        }
                        echo '.....Sonar scan completed'
                    }
                }
            }
        }
        
        stage("uploadNexus") {
            when { branch 'main'}
            steps {
                echo 'Uploading to nexus in progress.....'
                script {
                    lastStage = "${env.STAGE_NAME}"
                    pom = readMavenPom file: "pom.xml";
                    files = findFiles(glob: "build/*.${pom.packaging}");
                    artifactPath = files[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        nexusPublisher(
                            nexusInstanceId: NEXUS_INSTANCE_ID,
                            nexusRepositoryId: NEXUS_REPOSITORY,
                            packages: [
                                [
                                    $class: 'MavenPackage',
                                    mavenAssetList: [
                                        [classifier: '',
                                        extension: '',
                                        filePath: artifactPath]],
                                    mavenCoordinate:
                                        [artifactId: pom.artifactId,
                                        groupId: pom.groupId,
                                        packaging: pom.packaging,
                                        version: pom.version]
                                 ]
                            ]
                        )
                        echo '.....Artifact Uploaded successfully'
                    } else {
                        error "File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Nexus download & test') {
            when { branch 'main' }
                steps {
                    script { lastStage = "${env.STAGE_NAME}"}
                    script {
                        echo "Downloading artifact from nexus"
                        pom = readMavenPom file: "pom.xml";
                        groupId = pom.groupId;
                        groupIdPath = groupId.replace(".", "/");
                        sh """curl -X GET http://${env.NEXUS_SERVER}/repository/${env.NEXUS_REPOSITORY}/${groupIdPath}/${pom.artifactId}/${pom.version}/${pom.artifactId}-${pom.version}.${pom.packaging} -O"""
                    }
             }
        }
        stage("Tag Github") {
            when { branch 'main' }
            steps {
                script { lastStage = "${env.STAGE_NAME}"}
                withCredentials([[$class: 'UsernamePasswordMultiBinding', 
                    credentialsId: 'ms-iclab', 
                    usernameVariable: 'GIT_USERNAME', 
                    passwordVariable: 'GIT_PASSWORD']]) {
                        script {
                            pom = readMavenPom file: "pom.xml";
                            sh """git tag ${pom.version}"""
                            sh """git push ${pom.version} https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/isahub/ms-iclab.git --tags"""
                        }
                    }
            }
        }
    }
    post { 
        success {
            slackSend channel:"lab-ceres-mod4-sec1-status", message: "[Grupo6][Pipeline IC/CD][Rama: ${env.BRANCH_NAME}][Resultado: Éxito/Success]"
        }
        failure { 
            slackSend channel:"lab-ceres-mod4-sec1-status", message: "[Grupo6][Pipeline IC/CD][Rama: ${env.BRANCH_NAME}][Stage: ${lastStage}] [Resultado: Error/Fail]"
        }
    }
 }

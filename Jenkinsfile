pipeline {
    agent any
    stages {
        stage('compile') {
            steps {
                echo 'Source code compilation in progress.....'
                script {
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
        stage('sonarqube') {
            steps {
                echo 'Sonar scan in progress.....'
                script {
                    if(isUnix()) {
                        echo 'Unix OS'
                        sh './mvnw clean verify sonar:sonar \
                             -Dsonar.projectKey=ms-iclab \
                             -Dsonar.host.url=https://3818-186-175-87-17.sa.ngrok.io \
                             -Dsonar.login=sqp_69b753e9bbe8205404785182a4e8edd066c980bb'
                    } else {
                        echo 'Windows OS'
                        bat 'mvnw clean verify sonar:sonar \
                                -Dsonar.projectKey=ms-iclab \
                                -Dsonar.host.url=https://3818-186-175-87-17.sa.ngrok.io \
                                -Dsonar.login=sqp_69b753e9bbe8205404785182a4e8edd066c980bb'
                    }
                }
                echo '.....Sonar scan completed'
            }
        }
    }
}
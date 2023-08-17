pipeline {
    agent {
        
        docker {
            image 'maven:3.9.3-eclipse-temurin-17-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Clone Repository') {
            steps {
                script {
                    sh 'rm -rf .git'
                    final scmVars = checkout(scm)
                    env.BRANCH_NAME = scmVars.GIT_BRANCH
                    env.SHORT_COMMIT = "${scmVars.GIT_COMMIT[0..7]}"
                    env.GIT_REPO_NAME = scmVars.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
               }
           }
        }
        stage('Run Java Unit Tests') {
            steps {
                // sh 'mvn -B -DskipTests clean package'
                script {
                    withMaven(maven: 'M3') {
                        withEnv(["JAVA_HOME=/usr/java/jdk-17.0.5"])  {
                            sh "echo $JAVA_HOME"
                            // sh "mvn -Dmaven.test.failure.ignore=true -Dserver.port=8080 clean package"
                            sh "mvn -DskipTests clean package"
                        }
                    }
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
    }
}
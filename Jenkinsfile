pipeline {
    agent any
    stages {
        stage ('Build Backend'){
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }        
        stage ('Unit Tests'){
            steps {
                sh 'mvn test'
            }
        }
        // stage ('Sonar Analysis'){
        //     environment {
        //         scannerHome = tool 'SONAR_SCANER'
        //     }
        //     steps {
        //         withSonarQubeEnv('SONAR_LOCAL') {
        //             sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=67896846a13261bdc915fa671d42bfe6139ace82 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=src/test/**,**/model/**,**Application.java"
        //         }
        //     }
        // }
        // stage ('Quality Gate') {
        //     steps {
        //         sleep(10)
        //         timeout(time: 1, unit: 'MINUTES'){
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'Tomcat_Login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'Github_Login', url: 'https://github.com/ffvechio/tasks-api-tes'
                    sh 'mvn test'
                }
            }
        }
         stage ('Deploy Frontend') {
            steps {
                dir('frontend') { 
                    git credentialsId: 'Github_Login', url: 'https://github.com/ffvechio/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'Tomcat_Login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'Github_Login', url: 'https://github.com/ffvechio/tasks-functional-test'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Prod') {
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(10)
                dir('functional-test') {
                    sh 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports*.xml, functional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war, ', onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build $BUILD_NUMBER has failed', to: 'ffvechio+jenkins@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build $BUILD_NUMBER is fine!!', to: 'ffvechio+jenkins@gmail.com'
        }       
    }
}
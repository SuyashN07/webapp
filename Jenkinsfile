// pipeline {
//     agent none
    
//     stages {
//         stage('Build') {
//             agent any
//             tools {
//                 git 'git'
//             }
//             steps {
//                 bat 'mvn -B -DskipTests clean package'
//             }
//         }
// //         stage('Sonar-Report') {
// //             steps {
// //             sh 'mvn sonar:sonar \
// //   -Dsonar.projectKey=jenkins_project \
// //   -Dsonar.host.url=http://localhost:9000 \
// //   -Dsonar.login=5f09ded7e5db4d0ea0dcfd937c181af706e60475'
// //             }
// //         }
//         stage('Test') { 
//             agent any
//             tools {
//                 git 'git'
//             }
//             steps {
//                 bat 'mvn test' 
//             }
//             post {
//                 always {
//                     junit 'target/surefire-reports/*.xml' 
//                 }
//             }
//         }
//         stage('SonarQube Analysis') {
//             agent any
//             tools {
//                 git 'git'
//             }
//             steps {
//                 bat 'mvn clean install sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.token=sqa_c01df2e54ff5ebdb27de38d05c45d5162c8a1c2c'
//             }
//         }
//         stage('Deployment') {
//             agent { label 'slave_01' }
//             tools {
//                 git 'git-linux'
//             }
//             options {
//                 skipDefaultCheckout()
//             }
//             steps {
//                 sh '/home/lubuntu/deployment/deployment.sh'
//             }
//         }
//     }
// }

pipeline {
    agent any   // 👈 any free node

    environment {
        APP_NAME = "app.jar"
        PORT = "9999"
        SONAR_URL = "http://localhost:9000"
        SONAR_TOKEN = "sqa_c01df2e54ff5ebdb27de38d05c45d5162c8a1c2c"
    }

    stages {

        stage('Build') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn -B -DskipTests clean package'
                    } else {
                        bat 'mvn -B -DskipTests clean package'
                    }
                }
                stash includes: 'target/*.jar', name: 'app'
            }
        }

        stage('Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn test'
                    } else {
                        bat 'mvn test'
                    }
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Sonar') {
            steps {
                script {
                    if (isUnix()) {
                        sh "mvn sonar:sonar -Dsonar.host.url=$SONAR_URL -Dsonar.token=$SONAR_TOKEN"
                    } else {
                        bat "mvn sonar:sonar -Dsonar.host.url=%SONAR_URL% -Dsonar.token=%SONAR_TOKEN%"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                unstash 'app'

                script {
                    if (isUnix()) {
                        sh '''
                        set -e

                        mkdir -p deploy
                        cp target/*.jar deploy/$APP_NAME

                        echo "Stopping old app..."
                        pkill -f $APP_NAME || true

                        echo "Starting app..."
                        nohup java -jar deploy/$APP_NAME > deploy/app.log 2>&1 &

                        sleep 5
                        ss -tuln | grep $PORT || (echo "App failed to start" && exit 1)
                        '''
                    } else {
                        bat '''
                        if not exist deploy mkdir deploy
                        copy target\\*.jar deploy\\%APP_NAME%

                        echo Stopping old app...
                        taskkill /F /IM java.exe >nul 2>&1

                        echo Starting app...
                        start /B java -jar deploy\\%APP_NAME% > deploy\\app.log 2>&1

                        timeout /t 5 >nul
                        netstat -an | find "%PORT%" || (echo App failed to start & exit /b 1)
                        '''
                    }
                }
            }
        }
    }
}
pipeline {
    agent any

    tools {
        maven 'Maven 3'
        jdk   'jdk17'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh './mvnw compile -pl singleton'
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    steps {
                        sh './mvnw test -pl singleton'
                    }
                    post {
                        always {
                            junit 'singleton/target/surefire-reports/*.xml'
                        }
                    }
                }

                stage('Code Coverage - JaCoCo') {
                    steps {
                        sh './mvnw jacoco:report -pl singleton'
                    }
                    post {
                        always {
                            publishHTML([
                                reportDir:             'singleton/target/site/jacoco',
                                reportFiles:           'index.html',
                                reportName:            'JaCoCo Coverage Report',
                                keepAll:               true,
                                alwaysLinkToLastBuild: true,
                                allowMissing:          false
                            ])
                        }
                    }
                }

                stage('Code Quality') {
                    steps {
                        sh './mvnw checkstyle:checkstyle pmd:pmd -pl singleton'
                    }
                }

            }
        }

        stage('Documentation') {
            steps {
                sh './mvnw site -pl singleton'
            }
            post {
                always {
                    publishHTML([
                        reportDir:             'singleton/target/site',
                        reportFiles:           'index.html',
                        reportName:            'Maven Site + Javadoc',
                        keepAll:               true,
                        alwaysLinkToLastBuild: true,
                        allowMissing:          false
                    ])
                }
            }
        }

        stage('Package') {
            steps {
                sh './mvnw package -pl singleton -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'singleton/target/*.jar',
                                     fingerprint: true
                }
            }
        }

    }

    post {
        failure {
            mail to:      'ziyad@email.com',
                 subject: "ECHEC Pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body:    "Le build a echoue. Voir les logs ici : ${env.BUILD_URL}"
        }
        success {
            echo 'Pipeline termine avec succes!'
        }
    }
}
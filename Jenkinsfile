pipeline {
    agent any

    tools {
        maven 'Maven 3.5.2'   // nom configuré dans Jenkins > Global Tool Configuration
        jdk   'jdk11'
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

                stage('Tests Unitaires') {
                    steps {
                        sh './mvnw test -pl singleton'
                    }
                    post {
                        always {
                            junit 'singleton/target/surefire-reports/*.xml'
                        }
                    }
                }

                stage('Couverture de Code') {
                    steps {
                        sh './mvnw cobertura:cobertura -pl singleton'
                    }
                    post {
                        always {
                            publishHTML([
                                reportDir:   'singleton/target/site/cobertura',
                                reportFiles: 'index.html',
                                reportName:  'Cobertura Coverage'
                            ])
                        }
                    }
                }

                stage('Analyse Qualite') {
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
                        reportDir:   'singleton/target/site',
                        reportFiles: 'index.html',
                        reportName:  'Maven Site + Javadoc'
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

        stage('Deploy Nexus') {
            steps {
                sh './mvnw deploy -pl singleton -DskipTests'
            }
        }

    }

    post {
        failure {
            mail to:      'ziyad.khribach@esi.ac.ma',
                 subject: "ECHEC Pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body:    "Voir les logs ici : ${env.BUILD_URL}"
        }
        success {
            echo '✅ Pipeline terminé avec succès!'
        }
    }
}

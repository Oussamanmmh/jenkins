pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Récupération du code source"
                checkout scm
                sh 'chmod +x gradlew'
            }
        }

        stage('Test') {
            steps {
                echo "Phase Test : Lancement des tests unitaires"
                sh './gradlew clean test'

                echo "Archivage des résultats JUnit"
                junit allowEmptyResults: true, testResults: 'build/test-results/test/*.xml'

                echo "Génération des rapports Cucumber"
                script {
                    try {
                        sh './gradlew generateCucumberReports'
                        publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'build/reports/cucumber/html',
                            reportFiles: 'overview-features.html',
                            reportName: 'Cucumber Report'
                        ])
                    } catch (Exception e) {
                        echo "⚠️ Cucumber non généré : ${e.message}"
                    }
                }
            }
        }

        stage('Code Analysis') {
            steps {
                echo "Analyse du code avec SonarQube"
                withSonarQubeEnv('sonar') {
                    sh './gradlew sonar'
                }
            }
        }

        stage('Code Quality') {
            steps {
                echo "Vérification du Quality Gate"
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo "Génération du JAR et Javadoc"
                sh './gradlew jar javadoc'

                echo "Archivage du JAR"
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true

                echo "Archivage de la documentation"
                archiveArtifacts artifacts: 'build/docs/javadoc/**',
                                 fingerprint: true,
                                 allowEmptyArchive: true
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement vers le repository Maven"
                sh './gradlew publish'
            }
        }
    }

    post {

        always {
            echo "Fin du pipeline – nettoyage si nécessaire"
        }

        success {
            echo "Pipeline terminé avec succès"

            script {
                // EMAIL
                try {
                    emailext(
                        to: "lh_boulacheb@esi.dz",
                        subject: "Pipeline SUCCESS : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            <h2>Pipeline exécuté avec succès</h2>
                            <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                            <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                            <p><strong>Status :</strong> SUCCESS</p>
                            <p><a href="${env.BUILD_URL}">Voir le build</a></p>
                        """,
                        mimeType: 'text/html'
                    )
                } catch (e) {
                    echo "Erreur email : ${e.message}"
                }


                try {
                    slackSend(
                        channel: '#ogl',
                        color: 'good',
                        message: """
                         *Pipeline réussi*
                        *Projet:* ${env.JOB_NAME}
                        *Build:* #${env.BUILD_NUMBER}
                        *URL:* ${env.BUILD_URL}
                        """.stripIndent()
                    )
                } catch (e) {
                    echo "Erreur Slack : ${e.message}"
                }
            }
        }

        failure {
            echo "Pipeline échoué"

            script {
                try {
                    emailext(
                        to: "lh_boulacheb@esi.dz",
                        subject: "Pipeline FAILURE : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            <h2>Pipeline échoué</h2>
                            <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                            <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                            <p><strong>Status :</strong> FAILURE</p>
                            <p><a href="${env.BUILD_URL}console">Voir les logs</a></p>
                        """,
                        mimeType: 'text/html'
                    )
                } catch (e) {
                    echo "Erreur email : ${e.message}"
                }

                try {
                    slackSend(
                        channel: '#ogl',
                        color: 'danger',
                        message: """
                         *Pipeline échoué*
                        *Projet:* ${env.JOB_NAME}
                        *Build:* #${env.BUILD_NUMBER}
                        *Logs:* ${env.BUILD_URL}console
                        """.stripIndent()
                    )
                } catch (e) {
                    echo "Erreur Slack : ${e.message}"
                }
            }
        }

        unstable {
            echo "Pipeline instable"

            script {
                try {
                    emailext(
                        to: "lh_boulacheb@esi.dz",
                        subject: "Pipeline UNSTABLE : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            <h2>Pipeline instable</h2>
                            <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                            <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                            <p><strong>Status :</strong> UNSTABLE</p>
                            <p><a href="${env.BUILD_URL}">Voir le build</a></p>
                        """,
                        mimeType: 'text/html'
                    )
                } catch (e) {
                    echo "Erreur email : ${e.message}"
                }

                try {
                    slackSend(
                        channel: '#ogl',
                        color: 'warning',
                        message: """
                        *Pipeline instable*
                        *Projet:* ${env.JOB_NAME}
                        *Build:* #${env.BUILD_NUMBER}
                        *URL:* ${env.BUILD_URL}
                        """.stripIndent()
                    )
                } catch (e) {
                    echo "Erreur Slack : ${e.message}"
                }
            }
        }
    }
}

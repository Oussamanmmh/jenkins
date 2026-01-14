pipeline {

    agent any

    stages {

        stage('Test') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew test'
                junit 'build/test-results/test/*.xml'
                sh './gradlew cucumber'
            }
        }

//         stage('Code Analysis') {
//             steps {
//                 withSonarQubeEnv('sonar') {
//                     sh './gradlew sonar --info --stacktrace'
//                 }
//             }
//         }

//         stage('Quality Gate') {
//             steps {
//                 timeout(time: 1, unit: 'MINUTES') {
//                     waitForQualityGate abortPipeline: true
//                 }
//             }
//         }

        stage('Build') {
            steps {
                sh './gradlew build'
                sh './gradlew javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar'
                archiveArtifacts artifacts: 'build/docs/javadoc/**'
            }
        }

        stage('Deploy') {
            steps {
                sh './gradlew publish'
            }
        }
    }

    post {
        success {
            mail to: "oussamanmamcha@gmail.com",
                 subject: "Project deployed",
                 body: "Deployment success !!"
        }
        failure {
            mail to: "oussamanmamcha@gmail.com",
                 subject: "Project failed",
                 body: "Pipeline failed !!!!"
        }
    }
}

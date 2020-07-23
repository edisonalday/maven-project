pipeline {
    agent any
    tools {
        maven 'Maven'
        jdk 'Java'
    }
    stages{
        stage('Code Validation'){
            steps {
                sh 'mvn clean validate'
            }
        }
        stage('Code Compilation'){
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Static Code Analysis'){
            steps {
                sh 'mvn clean checkstyle:checkstyle'
                checkstyle canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'server/target/checkstyle-result.xml', unHealthy: ''
            }
        }
        stage('Unit Test'){
            steps {
                sh 'mvn clean test'
            }
            post {
                success { gerritReview score:1 }
                failure { gerritReview score:-1 }
            }
        }
        stage('Approval'){
            steps {
                timeout(time:5, unit:'DAYS'){
                    input message: 'Approve package?', ok: 'Approve', submitter: 'Edison Alday'
                }
            }
            post {
                success {
                    echo 'Packaged application code deployment to Nexus approved.'
                }
                failure {
                    echo 'Packaged application code deployment to Nexus failed - please visit test result.'
                }
            }
        }
        stage('Deploy to Nexus'){
            steps {
                sh 'mvn clean package deploy'
                nexusArtifactUploader artifacts: [[artifactId: 'maven-project', classifier: '', file: 'webapp/target/webapp.war', type: 'war']], credentialsId: 'nexusdeploymentrepo', groupId: 'com.example.maven-project', nexusUrl: 'nexus.dev-tools.gq:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }

        stage('Deploy to Development Server'){
            steps {
                deploy adapters: [tomcat9(credentialsId: 'Tomcat', path: '', url: 'http://stg.dev-tools.gq:8090')], contextPath: null, onFailure: false, war: '**/*.war'
                deploy adapters: [tomcat9(credentialsId: 'Tomcat', path: '', url: 'http://stg.dev-tools.gq:8090')], contextPath: null, onFailure: false, war: '**/*.war'
            }
            post {
                success {
                    echo 'Packaged successfully deployed to Development server.'
                }
                failure {
                    echo 'Packaged deployment failed.'
                }
            }
        }
    }
}
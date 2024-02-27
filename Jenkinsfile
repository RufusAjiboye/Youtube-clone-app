@Library('Jenkins_Shared_Library')

def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]


pipeline {
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    parameters {
        choice(name: 'Action', choices: 'Create\ndelete', description: 'Select create or destroy')
    }
    
    stages {
        stage('clean workspace') {
            steps{
                cleanWorkspace()
            }
        }
        
        
        stage('checkout from Git') {
            steps {
                checkoutGit('https://github.com/SoftwareDevDeveloper/Youtube-clone-app.git', 'main')
            }
        }
        
        stage('SonarQube Analysis') {
        when { expression { params.action == 'create'}}
            steps {
                sonarqubeAnalysis()
            }
            
        }
        
        stage('Sonar Quality gate') {
        when { expression { params.action == 'create'}}
            steps{
                script {
                    def credentialsId = 'sonar-token'
                    qualitygate(credentiasId)
                }
            }
        }
        
        stage('npm') {
        when { expression { params.action == 'create'}}
            steps {
                npmInstall()
            }
        }

        stage('OWASP FS SCAN') {
        when { expression { params.action == 'create'}}
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy file scan'){
        when { expression { params.action == 'create'}}
            steps{
                trivyFs()
            }
        }
    }
    
    post {
        always {
            echo 'Slack Notifications'
            slackSend (
                 channel: '#dummy',
                 color: COLOR_MAP[currentBuild.currentResult],
                 message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
           )
        }
    }
}
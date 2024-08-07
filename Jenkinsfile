pipeline {
agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/openjdk-21'
        PATH = "$JAVA_HOME/bin:$PATH"
    }
stages {
stage('Setup') {
            steps {
                script {
                    // Update package lists
                    sh 'apt-get update'
                    
                    // Install wget for downloading OpenJDK
                    sh 'apt-get install -y wget python3 python3-pip python3-venv'
                    // Create a virtual environment for Python
                    sh 'python3 -m venv ortelius-env'
                    
                    // Download and install OpenJDK 21
                    sh '''
                        curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
                        wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
                        tar -xvf jdk-21_linux-x64_bin.tar.gz
                        rm -rf /usr/lib/jvm
                        mkdir -p /usr/lib/jvm
                        mv jdk-21.0.3 /usr/lib/jvm/openjdk-21
                        update-alternatives --install /usr/bin/java java /usr/lib/jvm/openjdk-21/bin/java 1
                        update-alternatives --set java /usr/lib/jvm/openjdk-21/bin/java
                    '''

                    // Verify the Java installation
                    sh 'java -version'

                    // Install Maven
                    sh 'apt-get install -y maven'

                    // Verify Maven installation
                    sh 'mvn -version'
                }
            }
        }
stage('Checkout') {
steps {
// Checkout the code from the version control system
git url: 'https://github.com/indrajit75/webgoat.git', branch: 'main'
}
}

stage('Build') {
steps {
// Run Maven to build the JAR file
//sh 'apt-get update && apt-get install -y maven openjdk-21-ea+23_linux-x64_bin.tar.gz'
sh '''
mvn clean package
syft packages dir:$PWD -o cyclonedx-json > cyclonedx.json
. ortelius-env/bin/activate
pip install ortelius-cli
dh updatecomp --dhurl http://192.168.48.129 --dhuser admin --dhpass admin --rsp component.toml --deppkg "cyclonedx@cyclonedx.json"
jq -n --arg app "GLOBAL.MedImpact Healthcare Systems Inc.DevSecOps.WebGoat;1_0_0" --arg env "GLOBAL.MedImpact Healthcare Systems Inc.DevSecOps.Development" '{application: $app, environment: $env,  skipdeploy: "Y", rc: 0 }' > deploy.json
dh deploy --dhurl http://192.168.48.129 --dhuser admin --dhpass admin --logdeployment --deploydata deploy.json --appautoinc N
deactivate
'''
}
}

stage('SonarQube Analysis') {
            environment {
                // Set up SonarQube environment variables
                SONARQUBE_URL = 'http://192.168.48.129:9000'
                SONARQUBE_TOKEN = 'sqp_58654163ae3e61cf1c304fc3167264922f93dec8'
            }
            steps {
                script {
                    // Run SonarQube scanner
                    sh '''
                        mvn sonar:sonar -Dsonar.projectKey=webgoat \
                                        -Dsonar.host.url=$SONARQUBE_URL \
                                        -Dsonar.token=$SONARQUBE_TOKEN
                    '''
                }
            }
        }
    
stage('Archive') {
steps {
// Archive the JAR file as a build artifact
archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
}
}
}

post {
// Post actions that always run after the pipeline, such as cleanup or notifications
always {
// Clean up workspace
cleanWs()
}
}
}

pipeline {
    agent any

    environment {
        MAVEN_OPTS = '-Dmaven.test.skip=false'
    }

    stages {
        // ------------------------------------
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/werteninadra/bnahabilitations.git'
            }
        }

        // ------------------------------------
        stage('Build Backend') {
            steps {
                dir('habilitationbna') {
                    sh 'mvn clean compile'
                }
            }
        }

        // ------------------------------------
        stage('Run Tests Backend') {
            steps {
                dir('habilitationbna') {
                    sh 'mvn test'
                }
            }
        }

        // ------------------------------------
        stage('JaCoCo Coverage') {
            steps {
                dir('habilitationbna') {
                    sh 'mvn jacoco:report'
                    jacoco execPattern: '**/target/jacoco*.exec',
                           classPattern: '**/target/classes',
                           sourcePattern: '**/src/main/java'
                }
            }
        }

        // ------------------------------------
        stage('SonarQube Analysis') {
            environment {
                SONAR_HOST_URL = 'http://localhost:9000'
                SONAR_LOGIN = credentials('sonar-token')
            }
            steps {
                dir('habilitationbna') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=habilitationbna \
                          -Dsonar.projectName="habilitationbna" \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.login=$SONAR_LOGIN
                    '''
                }
            }
        }

        // ------------------------------------
        stage('Deploy to Nexus') {
            steps {
                dir('habilitationbna') {
                    sh 'mvn deploy -Dmaven.test.skip=true'
                }
            }
        }

     // ------------------------------------
stage('Build Docker Images via Compose') {
    steps {
        sh 'docker compose -f docker-compose.yml build'
    }
}

// ------------------------------------
stage('Docker Login') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
    }
}

// ------------------------------------
stage('Push Docker Images') {
    steps {
        script {
            sh 'docker tag habilitationbna nadrawertani/habilitationbna:latest'
            sh 'docker tag foyer-management-frontend nadrawertani/foyer-management-frontend:latest'
            sh 'docker push nadrawertani/habilitationbna:latest'
            sh 'docker push nadrawertani/foyer-management-frontend:latest'
        }
    }
}

        // ------------------------------------
        stage('Grafana Metrics') {
            steps {
                script {
                    def status = currentBuild.currentResult == 'SUCCESS' ? 1 : 0
                    sh """
                        cat <<EOF | curl --data-binary @- http://localhost:9090/metrics/job/${env.JOB_NAME}/build/${env.BUILD_NUMBER}
                        # TYPE jenkins_build_status gauge
                        jenkins_build_status{job="${env.JOB_NAME}",build="${env.BUILD_NUMBER}"} ${status}
                        EOF
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Build finished with status: ${currentBuild.currentResult}"
        }
    }
}

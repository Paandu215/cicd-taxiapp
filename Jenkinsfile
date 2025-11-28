def registry = 'https://trial22bjqq.jfrog.io/artifactory'
def imageName = 'trial22bjqq.jfrog.io/taxi01-docker-local/taxiapp'
def version   = '1.0.1'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.11/bin:$PATH"
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

    stages {

        stage("build") {
            steps {
                echo "----------- build started ----------"
                sh 'mvn package'
                echo "----------- build completed ----------"
            }
        }

        stage("test") {
            steps {
                echo "----------- unit test started ----------"
                sh 'mvn surefire-report:report'
                echo "----------- unit test Completed ----------"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    sh """
                    mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \
                    -Dsonar.projectKey=taxi-app1234_taxi \
                    -Dsonar.organization=taxi-app1234 \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.token=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'

                    def server = Artifactory.newServer(
                        url: registry,
                        credentialsId: "jfrog-cred"
                    )

                    def properties = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*",
                                "target": "taxi-libs-release-local/",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)

                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }

        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    def app = docker.build("${imageName}:${version}")
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'

                    docker.withRegistry(registry, 'jfrog-cred') {
                        app.push()
                    }

                    echo '<--------------- Docker Publish Ended --------------->'
                }
            }
        }

        stage("Deploy") {
            steps {
                script {
                    sh './deploy.sh'
                }
            }
        }
    }
}

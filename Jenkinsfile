pipeline {
    agent {
        label "Jenkins-Slave"
    }

    environment {
        APP_NAME = "reddit-clone-pipeline"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("SCM Checkout") {
            steps {
                git branch: 'main',
                    credentialsId: 'GitHub-Token',
                    url: 'https://github.com/yomesky2000/a-reddit-clone-gitops'
                echo "Continuous Deployment GitOps Repo Cloned Successfully"
            }
        }

        stage("Get Latest Docker Image Tag") {
            steps {
                script {
                    def imageName = "ginger2000/reddit-clone-pipeline"
                    def response = sh(
                        script: """curl -s https://hub.docker.com/v2/repositories/${imageName}/tags?page_size=1 | jq -r '.results[0].name'""",
                        returnStdout: true
                    ).trim()

                    if (response) {
                        env.IMAGE_TAG = response
                        echo "Fetched latest Docker image tag: ${env.IMAGE_TAG}"
                    } else {
                        echo "Failed to fetch image tag. Falling back to Jenkins BUILD_NUMBER: ${env.IMAGE_TAG}"
                    }
                }
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    echo "Using APP_NAME: ${APP_NAME}"
                    echo "Using IMAGE_TAG: ${IMAGE_TAG}"

                    echo "Before updating image tag:"
                    cat deployment.yaml

                    sed -i "s|${APP_NAME}:.*|${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml

                    echo "After updating image tag:"
                    cat deployment.yaml
                """
                echo "Container tags updated successfully"
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                script {
                    sh """
                        git config --global user.name "Ginger"
                        git config --global user.email "ginger0@gmail.com"
                        git add deployment.yaml || true
                    """

                    // Only commit and push if there are changes
                    def hasChanges = sh(
                        script: 'git diff --cached --quiet || echo "changed"',
                        returnStdout: true
                    ).trim()

                    if (hasChanges == "changed") {
                        sh "git commit -m 'Updated Deployment Manifest to ${IMAGE_TAG}'"

                        withCredentials([usernamePassword(
                            credentialsId: 'GitHub-Token',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )]) {
                            sh """
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/yomesky2000/a-reddit-clone-gitops main
                            """
                            echo "Pushed updated deployment.yaml to GitHub"
                        }
                    } else {
                        echo "No changes to commit."
                    }
                }
            }
        }
    }
}

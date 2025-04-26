pipeline {
    agent {
        label "Jenkins-Slave"
    }

    environment {
        APP_NAME = "reddit-clone-pipeline"
        DOCKER_USER = "ginger2000"
        VERSION = "1.0.0"
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
                echo "GitOps Repo Cloned Successfully"
            }
        }

        stage("Prepare Image Tag") {
            steps {
                script {
                    env.IMAGE_TAG = "${VERSION}-${BUILD_NUMBER}"
                    env.FULL_IMAGE = "${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
                    echo "Full Docker image tag: ${env.FULL_IMAGE}"
                }
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    echo "Before updating deployment.yaml:"
                    cat deployment.yaml

                    # Replace the line that starts with 'image: ' with the new image tag
                    sed -i "s|image: .*|image: ${FULL_IMAGE}|g" deployment.yaml

                    echo "After updating deployment.yaml:"
                    cat deployment.yaml
                """
                echo "Deployment manifest updated successfully"
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
                        sh "git commit -m 'Updated Deployment Manifest to ${FULL_IMAGE}'"

                        withCredentials([usernamePassword(
                            credentialsId: 'GitHub-Token',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )]) {
                            sh """
                                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/yomesky2000/a-reddit-clone-gitops main
                            """
                            echo "Changes pushed to GitHub"
                        }
                    } else {
                        echo "No changes to commit."
                    }
                }
            }
        }
    }
}

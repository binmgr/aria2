// Jenkins declarative pipeline for the aria2 static binary builder.
// Verification build: mirrors the gates of GitHub Actions, Gitea/Forgejo, and GitLab CI.
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 4, unit: 'HOURS')
    }

    environment {
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'binmgr/aria2'
    }

    stages {
        stage('Get aria2 version') {
            steps {
                script {
                    if (env.ARIA2_VERSION_OVERRIDE?.trim()) {
                        env.ARIA2_VERSION = env.ARIA2_VERSION_OVERRIDE
                    } else {
                        env.ARIA2_VERSION = sh(
                            returnStdout: true,
                            script: """curl -fsSL https://api.github.com/repos/aria2/aria2/releases/latest | jq -r '.tag_name' | sed 's/^release-//'"""
                        ).trim()
                    }
                    echo "Building aria2 ${env.ARIA2_VERSION}"
                }
            }
        }

        stage('Security') {
            parallel {
                stage('Secret scan') {
                    agent {
                        docker {
                            image 'trufflesecurity/trufflehog:latest'
                            args '--entrypoint=""'
                        }
                    }
                    steps {
                        sh 'trufflehog git file://. --since-commit HEAD~1 --fail --only-verified'
                    }
                }
            }
        }

        stage('Build binaries') {
            matrix {
                axes {
                    axis {
                        name 'TARGET'
                        values 'linux-amd64', 'linux-arm64', 'windows-amd64', 'windows-arm64', 'darwin-amd64', 'darwin-arm64', 'freebsd-amd64', 'freebsd-arm64'
                    }
                }
                stages {
                    stage('Build') {
                        agent {
                            docker {
                                image "${env.REGISTRY}/${env.IMAGE_NAME}:build"
                                reuseNode true
                            }
                        }
                        steps {
                            sh "build-aria2 ${TARGET} ${ARIA2_VERSION} out"
                            archiveArtifacts artifacts: 'out/**', allowEmptyArchive: false
                        }
                    }
                }
            }
        }

        stage('Release') {
            when { buildingTag() }
            steps {
                echo 'Release publishing is performed by the hosted provider (GitHub/GitLab/Gitea/Forgejo) — Jenkins build is verification-only.'
            }
        }
    }
}

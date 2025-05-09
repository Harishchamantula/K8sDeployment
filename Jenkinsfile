pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Harishchamantula/K8sDeployment.git'
        CREDENTIALS_ID = 'Tokenssi2'
    }

    stages {
        stage('Set Branch Info') {
            steps {
                script {
                    BRANCH_NAME = env.BRANCH_NAME ?: 'Dev'
                    echo "Branch detected: ${BRANCH_NAME}"

                    env.TARGET_BRANCH = BRANCH_NAME
                    env.COMPARE_BRANCH = (BRANCH_NAME == 'Prod') ? 'origin/Dev' : 'origin/Prod'
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.TARGET_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${REPO_URL}",
                        credentialsId: "${CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Detect Changed Chart Folders') {
            steps {
                script {
                    sh "git fetch origin"
                    sh "git diff --name-only ${env.COMPARE_BRANCH}...HEAD > changed_files.txt"

                    def changedFiles = readFile('changed_files.txt')
                        .split('\n')
                        .collect { it.trim() }
                        .findAll { it && fileExists(it) && it.contains("deploy_config.yaml") }

                    def chartFolders = changedFiles.collect { file ->
                        return file.substring(0, file.lastIndexOf("/"))
                    }.unique()

                    if (chartFolders.isEmpty()) {
                        echo "No chart folders changed."
                        currentBuild.result = 'SUCCESS'
                        return
                    }

                    env.CHANGED_CHARTS = chartFolders.join(',')
                    echo "Changed chart folders: ${env.CHANGED_CHARTS}"
                }
            }
        }

        stage('Deploy Changed Charts with Helm') {
            when {
                expression { return env.CHANGED_CHARTS?.trim() }
            }
            steps {
                script {
                    def charts = env.CHANGED_CHARTS.split(',')

                    charts.each { chartPath ->
                        def configFile = "${chartPath}/deploy_config.yaml"

                        if (!fileExists(configFile)) {
                            echo "❌ Skipping ${chartPath}: No deploy_config.yaml"
                            return
                        }

                        def config = readYaml file: configFile
                        def releaseName = config.releaseName
                        def namespace = config.namespace

                        echo """
                        🧭 Deploying chart from: ${chartPath}
                        - Namespace: ${namespace}
                        - Release: ${releaseName}
                        """

                        // Ensure namespace exists
                        sh "kubectl get ns ${namespace} || kubectl create ns ${namespace}"

                        // Deploy with Helm
                        sh """
                            helm upgrade --install ${releaseName} ${chartPath} \
                            -f ${chartPath}/values.yaml \
                            --namespace ${namespace}
                        """
                    }
                }
            }
        }
    }
}

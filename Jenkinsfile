def img_build_hook = null

pipeline {
    stages {
        stage('Check Image Build Hook') {
            when {
                expression { env.IMG_BUILD_HOOKS != null }
            }
            steps {
                echo "Check Image Build Hook"
                script {
                    def jsonObj = readJSON text: env.IMG_BUILD_HOOKS
                    if (env.GIT_URL in jsonObj) {
                        echo "Build docker image"
                        if (env.BRANCH_NAME in jsonObj[env.GIT_URL]) {
                            img_build_hook = jsonObj[env.GIT_URL][env.BRANCH_NAME]
                        } else {
                            img_build_hook = jsonObj[env.GIT_URL]['default']
                        }
                    }
                }
            }
        }
        stage('Build & Push Image') {
            when {
                allOf {
                    expression { img_build_hook != null }
                    expression { env.CHANGE_ID == null } // Not pull request
                }
            }
            steps {
                script {
                    echo "Build docker image"
                    sh """cat <<EOF > payload_file.yaml
env:
   - name: "tarball_url"
     value: "${tarball_url}"
EOF"""
                    sh "curl -i -H 'Content-Type: application/yaml' --data-binary @payload_file.yaml -k -X POST ${img_build_hook}"
                }
            }
        }
    }
}

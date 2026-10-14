pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 20, unit: 'MINUTES')
    }

    environment {
        APP_NAME   = 'LW-Claude-portal'
        DEPLOY_DIR = '/var/www/html'          // prod web root (Apache httpd / nginx)
        APP_URL    = 'http://localhost'       // URL used for the live smoke test
        CLAUDE_BIN = '/usr/local/bin/claude'
    }

    stages {

        stage('01 - Create HTML Code') {
            steps {
                sh '''
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>CloudOps Status Portal</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            background: #0f172a;
            color: #e2e8f0;
            margin: 0;
            padding: 50px 20px;
            text-align: center;
        }

        main {
            max-width: 650px;
            margin: auto;
            padding: 35px;
            background: #1e293b;
            border-radius: 18px;
        }

        .status {
            color: #4ade80;
            font-size: 24px;
            font-weight: bold;
        }

        .tag {
            color: #93c5fd;
        }
    </style>
</head>

<body>
    <main>
        <p class="tag">LW Claude With Devops</p>

        <h1>CloudOps Status Portal</h1>

        <p class="status">SERVICE OPERATIONAL</p>

        <p>Application: LW-Claude-portal</p>
        <p>Version: 1.0.0</p>
        <p>Deployment: Jenkins CI/CD</p>

        <p id="health">Health check: Ready</p>
    </main>
</body>
</html>
EOF
echo "index.html created:"
ls -l index.html
'''
            }
        }

        stage('02 - Claude AI To Review Code') {
            steps {
                aiAgent(
                    agent: claudeCode(),
                    executablePath: "${env.CLAUDE_BIN}",
                    prompt: '''
You are a senior DevOps and application engineer.

Review index.html in the current workspace.

Explain:
1. What this app does
2. Any vulnerable security issue or concern
3. Whether the HTML looks correct
4. Any improvement that needs to be done

Use very simple English.

Do not edit anything yet.
Do not expose secrets and passwords.
'''
                )
            }
        }

        stage('03 - Automated Quality Gate') {
            steps {
                sh '''
set -e
echo "Checking index.html exists and is not empty..."
test -s index.html

echo "Checking required content..."
grep -q '<!DOCTYPE html>'           index.html
grep -q '<title>CloudOps Status Portal</title>' index.html
grep -q 'LW Claude With Devops'     index.html
grep -q 'SERVICE OPERATIONAL'       index.html
grep -q '</html>'                   index.html

echo "Quality gate PASSED"
'''
            }
        }

        stage('04 - Deployment Approval') {
            steps {
                input(
                    message: 'Approve deployment to prod env?',
                    ok: 'Approve prod env'
                )
            }
        }

        stage('05 - Deploy To Prod Env') {
            steps {
                sh '''
set -e
echo "Deploying ${APP_NAME} to ${DEPLOY_DIR} ..."
sudo mkdir -p "${DEPLOY_DIR}"
sudo cp index.html "${DEPLOY_DIR}/index.html"
sudo chmod 644 "${DEPLOY_DIR}/index.html"
echo "Deployment done"
ls -l "${DEPLOY_DIR}/index.html"
'''
            }
        }

        stage('06 - Live Smoke Test') {
            steps {
                sh '''
set -e
echo "Running smoke test against ${APP_URL} ..."
for i in 1 2 3 4 5; do
    if curl -fsS "${APP_URL}" | grep -q 'SERVICE OPERATIONAL'; then
        echo "Smoke test PASSED - app is live"
        exit 0
    fi
    echo "Attempt $i failed, retrying in 5s..."
    sleep 5
done
echo "Smoke test FAILED - app not reachable or content wrong"
exit 1
'''
            }
        }

        stage('07 - Claude Deployment Summary') {
            steps {
                aiAgent(
                    agent: claudeCode(),
                    executablePath: "${env.CLAUDE_BIN}",
                    prompt: '''
You are a DevOps release engineer.

Review the Jenkins workspace file index.html.

1. Summarize the app and its expected purpose
2. Summarize this release: app name, version, deployment method
3. Confirm the quality gate and live smoke test passed
4. List any risks or follow-up actions for the next release

Use very simple English.
Do not edit anything.
Do not expose secrets and passwords.
'''
                )
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'index.html', allowEmptyArchive: true
        }
        success {
            echo 'Pipeline SUCCESS - LW-Claude-portal deployed to prod'
        }
        failure {
            echo 'Pipeline FAILED - check the stage logs above'
        }
    }
}

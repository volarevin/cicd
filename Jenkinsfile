pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'repo link'
        GIT_CREDENTIALS_ID = 'github-pat'
        GIT_BRANCH = 'main'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${env.GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${env.GIT_REPO_URL}",
                        credentialsId: "${env.GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                sudo rsync -av --delete \
                --exclude='venv/' \
                --exclude='.git/' \
                ./ /var/www/html/

                sudo chown -R www-data:www-data /var/www/html/
                '''
            }
        }

    }
}

7. Audit Logic (test.py)
This Python script uses Selenium to verify that the server successfully renders the PHP without exposing raw code or throwing 500-level errors.
import os
import sys
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service

test_file = os.getenv("TARGET_PHP_FILE", "index.php")

options = Options()
options.add_argument("--headless=new")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")

service = Service("/usr/bin/chromedriver")
driver = webdriver.Chrome(service=service, options=options)

try:
    url = f"http://localhost/staging/{test_file}"
    print(f"🚀 Auditing: {url}")
    
    driver.get(url)
    content = driver.page_source
    lower_content = content.lower()

    # 1. HARD BLOCK: Check for PHP Error Strings
    errors = ["fatal error", "parse error", "warning:", "stack trace:", "xdebug-error"]
    found_errors = [e for e in errors if e in lower_content]
    
    if found_errors:
        print(f"❌ DEPLOYMENT BLOCKED: Found PHP errors: {found_errors}")
        sys.exit(1)

    # 2. HARD BLOCK: Check for Raw PHP Leakage
    # If "<?php" appears in the browser, the server failed to process the file.
    if "<?php" in content or "<?=" in content:
        print("❌ DEPLOYMENT BLOCKED: Raw PHP code leaked! Apache is not executing PHP.")
        sys.exit(1)

    # 3. SOFT BLOCK: Check for Empty Body
    # If the page has 0 text content, it's likely a silent crash (WSOD - White Screen of Death)
    body_text = driver.find_element("tag name", "body").text.strip()
    if not body_text and "img" not in lower_content:
        print("❌ DEPLOYMENT BLOCKED: Page is blank. Possible silent PHP crash.")
        sys.exit(1)

    print(f"✅ PASS: {test_file} rendered without errors.")

except Exception as e:
    print(f"⚠️ TEST SYSTEM ERROR: {e}")
    sys.exit(1)
finally:
    driver.quit()

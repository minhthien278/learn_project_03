pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.6'
        jdk 'JDK 17'
    }
    
    environment {
        // Docker Hub credentials - sử dụng username/password credential
        DOCKERHUB = credentials('dockerhub-credentials')
        
        // Define service paths for easier reference
        CONFIG_SERVER_PATH = "spring-petclinic-config-server"
        DISCOVERY_SERVER_PATH = "spring-petclinic-discovery-server"
        ADMIN_SERVER_PATH = "spring-petclinic-admin-server"
        API_GATEWAY_PATH = "spring-petclinic-api-gateway"
        CUSTOMERS_SERVICE_PATH = "spring-petclinic-customers-service"
        VISITS_SERVICE_PATH = "spring-petclinic-visits-service"
        VETS_SERVICE_PATH = "spring-petclinic-vets-service"
        GENAI_SERVICE_PATH = "spring-petclinic-genai-service"
    }
    
    parameters {
        booleanParam(
            name: 'FORCE_BUILD_ALL',
            defaultValue: false,
            description: 'Force build all services regardless of changes'
        )
        booleanParam(
            name: 'SKIP_DOCKER_BUILD',
            defaultValue: false,
            description: 'Skip Docker build and push (for testing only)'
        )
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh "git fetch --all"
            }
        }

        stage('🚀 Initialize') {
            steps {
                script {
                    echo "🎯 Starting CI Pipeline for PetClinic Microservices"
                    echo "📍 Branch: ${env.BRANCH_NAME}"
                    echo "🔨 Build Number: ${env.BUILD_NUMBER}"

                    env.COMMIT_ID       = sh(returnStdout: true,
                                            script: 'git rev-parse --short HEAD').trim()
                    env.COMMIT_MESSAGE  = sh(returnStdout: true,
                                            script: 'git log -1 --pretty=%B').trim()

                    def tag = env.TAG_NAME?.trim()?.toLowerCase()
                    env.IS_TAG_BUILD = (tag && tag != 'null')

                    if (env.IS_TAG_BUILD) {
                        env.PRIMARY_TAG   = env.TAG_NAME          // e.g. v1.2.3
                        env.SECONDARY_TAG = ''                    // not needed
                        echo "🏷️  Detected tag build: ${env.TAG_NAME}"
                    } else {
                        // Normal branch / PR build (your original logic)
                        if (env.BRANCH_NAME == 'main') {
                            env.PRIMARY_TAG   = 'latest'
                            env.SECONDARY_TAG = env.COMMIT_ID
                        } else {
                            env.PRIMARY_TAG   = "${env.BRANCH_NAME}-${env.COMMIT_ID}".replace('/', '-')
                            env.SECONDARY_TAG = env.BRANCH_NAME.replace('/', '-')
                        }
                    }

                    /* ---------- 3.  DIAGNOSTICS ---------- */
                    echo '=== Build Information ==='
                    echo "📍 Ref type   : ${env.IS_TAG_BUILD == true ? 'TAG' : 'BRANCH'}"
                    echo "🔖 Commit ID  : ${env.COMMIT_ID}"
                    echo "🏷️  Primary   : ${env.PRIMARY_TAG}"
                    echo "🏷️  Secondary : ${env.SECONDARY_TAG}"
                    echo "💬 Commit msg : ${env.COMMIT_MESSAGE}"
                }
            }
        }
        
        stage('Detect Changes') {
            steps {
                script {
                    // Initialize change flags - default to false
                    env.CONFIG_SERVER_CHANGED = "false"
                    env.DISCOVERY_SERVER_CHANGED = "false"
                    env.ADMIN_SERVER_CHANGED = "false"
                    env.API_GATEWAY_CHANGED = "false"
                    env.CUSTOMERS_SERVICE_CHANGED = "false"
                    env.VISITS_SERVICE_CHANGED = "false"
                    env.VETS_SERVICE_CHANGED = "false"
                    env.GENAI_SERVICE_CHANGED = "false"
                    
                    // For first build or when specifically requested, build everything
                    if (params.FORCE_BUILD_ALL == true || currentBuild.number == 1) {
                        echo "Building all services (first build or forced build)"
                        env.CONFIG_SERVER_CHANGED = "true"
                        env.DISCOVERY_SERVER_CHANGED = "true"
                        env.ADMIN_SERVER_CHANGED = "true"
                        env.API_GATEWAY_CHANGED = "true"
                        env.CUSTOMERS_SERVICE_CHANGED = "true"
                        env.VISITS_SERVICE_CHANGED = "true"
                        env.VETS_SERVICE_CHANGED = "true"
                        env.GENAI_SERVICE_CHANGED = "true"
                    } else {
                        try {
                            // Get the current commit hash
                            def currentCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                            
                            // Get the previous successful build's commit
                            def previousCommit = ""
                            if (currentBuild.previousSuccessfulBuild) {
                                previousCommit = sh(script: "git rev-parse HEAD~1", returnStdout: true).trim()
                            } else {
                                echo "No previous successful build found, comparing with previous commit"
                                previousCommit = sh(script: "git rev-parse HEAD~1", returnStdout: true).trim()
                            }
                            
                            echo "Comparing changes between ${previousCommit} and ${currentCommit}"
                            
                            // Get the changed files between the two commits
                            def changeSet = sh(script: "git diff --name-only ${previousCommit} ${currentCommit}", returnStdout: true).trim()
                            
                            if (changeSet) {
                                changeSet = changeSet.split('\n')
                                echo "Changed files: ${changeSet.join(', ')}"
                                

                                // Check each path in the change set
                                changeSet.each { change ->
                                    if (change.startsWith(CONFIG_SERVER_PATH)) {
                                        env.CONFIG_SERVER_CHANGED = "true"
                                        echo "Config Server changes detected"
                                    }
                                    else if (change.startsWith(DISCOVERY_SERVER_PATH)) {
                                        env.DISCOVERY_SERVER_CHANGED = "true" 
                                        echo "Discovery Server changes detected"
                                    }
                                    else if (change.startsWith(ADMIN_SERVER_PATH)) {
                                        env.ADMIN_SERVER_CHANGED = "true"
                                        echo "Admin Server changes detected"
                                    }
                                    else if (change.startsWith(API_GATEWAY_PATH)) {
                                        env.API_GATEWAY_CHANGED = "true"
                                        echo "API Gateway changes detected"
                                    }
                                    else if (change.startsWith(CUSTOMERS_SERVICE_PATH)) {
                                        env.CUSTOMERS_SERVICE_CHANGED = "true"
                                        echo "Customers Service changes detected"
                                    }
                                    else if (change.startsWith(VISITS_SERVICE_PATH)) {
                                        env.VISITS_SERVICE_CHANGED = "true"
                                        echo "Visits Service changes detected"
                                    }
                                    else if (change.startsWith(VETS_SERVICE_PATH)) {
                                        env.VETS_SERVICE_CHANGED = "true"
                                        echo "Vets Service changes detected"
                                    }
                                    else if (change.startsWith(GENAI_SERVICE_PATH)) {
                                        env.GENAI_SERVICE_CHANGED = "true"
                                        echo "GenAI Service changes detected"
                                    }
                                }
                                
                                // Handle infrastructure service changes that would affect other services
                                if (env.CONFIG_SERVER_CHANGED == "true") {
                                    echo "Config Server changes may affect all services"
                                    // Optionally build all services if config server changes
                                }
                            } else {
                                echo "No changes detected or git diff returned empty result"
                                currentBuild.result = 'ABORTED'
                                error("No changes detected to build")
                            }
                        } catch (Exception e) {
                            echo "Error detecting changes: ${e.message}"
                            echo "Building all services as fallback"
                            env.CONFIG_SERVER_CHANGED = "true"
                            env.DISCOVERY_SERVER_CHANGED = "true"
                            env.ADMIN_SERVER_CHANGED = "true"
                            env.API_GATEWAY_CHANGED = "true"
                            env.CUSTOMERS_SERVICE_CHANGED = "true"
                            env.VISITS_SERVICE_CHANGED = "true"
                            env.VETS_SERVICE_CHANGED = "true"
                            env.GENAI_SERVICE_CHANGED = "true"
                        }
                    }
                    
                    // Output summary of what will be built
                    echo "Services to build:"
                    echo "Config Server: ${env.CONFIG_SERVER_CHANGED}"
                    echo "Discovery Server: ${env.DISCOVERY_SERVER_CHANGED}"
                    echo "Admin Server: ${env.ADMIN_SERVER_CHANGED}" 
                    echo "API Gateway: ${env.API_GATEWAY_CHANGED}"
                    echo "Customers Service: ${env.CUSTOMERS_SERVICE_CHANGED}"
                    echo "Visits Service: ${env.VISITS_SERVICE_CHANGED}"
                    echo "Vets Service: ${env.VETS_SERVICE_CHANGED}"
                    echo "GenAI Service: ${env.GENAI_SERVICE_CHANGED}"
                }
            }
        }
        
        stage('Build') {
            parallel {
                stage('Config Server') {
                    when {
                        expression { return env.CONFIG_SERVER_CHANGED == "true" }
                    }
                    steps {
                        dir(CONFIG_SERVER_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "Config Server built"
                        }
                    }
                }
                
                stage('Discovery Server') {
                    when {
                        expression { return env.DISCOVERY_SERVER_CHANGED == "true" }
                    }
                    steps {
                        dir(DISCOVERY_SERVER_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "Discovery Server built"
                        }
                    }
                }
                
                stage('Admin Server') {
                    when {
                        expression { return env.ADMIN_SERVER_CHANGED == "true" }
                    }
                    steps {
                        dir(ADMIN_SERVER_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "Admin Server built"
                        }
                    }
                }
                
                stage('API Gateway') {
                    when {
                        expression { return env.API_GATEWAY_CHANGED == "true" }
                    }
                    steps {
                        dir(API_GATEWAY_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "API Gateway built"
                        }
                    }
                }
                
                stage('Customers Service') {
                    when {
                        expression { return env.CUSTOMERS_SERVICE_CHANGED == "true" }
                    }
                    steps {
                        dir(CUSTOMERS_SERVICE_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "Customers Service built"
                        }
                    }
                }
                
                stage('Visits Service') {
                    when {
                        expression { return env.VISITS_SERVICE_CHANGED == "true" }
                    }
                    steps {
                        dir(VISITS_SERVICE_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "Visits Service built"
                        }
                    }
                }
                
                stage('Vets Service') {
                    when {
                        expression { return env.VETS_SERVICE_CHANGED == "true" }
                    }
                    steps {
                        dir(VETS_SERVICE_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "Vets Service built"
                        }
                    }
                }
                
                stage('GenAI Service') {
                    when {
                        expression { return env.GENAI_SERVICE_CHANGED == "true" }
                    }
                    steps {
                        dir(GENAI_SERVICE_PATH) {
                            sh 'mvn clean package -DskipTests'
                            echo "GenAI Service built"
                        }
                    }
                }
            }
        }
        
        
        stage('🐳 Docker Build & Push') {
            when {
                expression { return params.SKIP_DOCKER_BUILD != true }
            }
            steps {
                script {
                    echo "🔐 Logging into Docker Hub..."
                    sh '''
                        echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                    '''
                    
                    // Define service mappings
                    def services = [
                        [path: CONFIG_SERVER_PATH, changed: env.CONFIG_SERVER_CHANGED, name: 'spring-petclinic-config-server'],
                        [path: DISCOVERY_SERVER_PATH, changed: env.DISCOVERY_SERVER_CHANGED, name: 'spring-petclinic-discovery-server'],
                        [path: ADMIN_SERVER_PATH, changed: env.ADMIN_SERVER_CHANGED, name: 'spring-petclinic-admin-server'],
                        [path: API_GATEWAY_PATH, changed: env.API_GATEWAY_CHANGED, name: 'spring-petclinic-api-gateway'],
                        [path: CUSTOMERS_SERVICE_PATH, changed: env.CUSTOMERS_SERVICE_CHANGED, name: 'spring-petclinic-customers-service'],
                        [path: VISITS_SERVICE_PATH, changed: env.VISITS_SERVICE_CHANGED, name: 'spring-petclinic-visits-service'],
                        [path: VETS_SERVICE_PATH, changed: env.VETS_SERVICE_CHANGED, name: 'spring-petclinic-vets-service'],
                        [path: GENAI_SERVICE_PATH, changed: env.GENAI_SERVICE_CHANGED, name: 'spring-petclinic-genai-service']
                    ]
                    
                    def buildResults = [:]
                    def successCount = 0
                    
                    services.each { service ->
                        if (service.changed == "true") {
                            echo ""
                            echo "=" * 60
                            echo "🔨 Building Docker image for: ${service.name}"
                            echo "=" * 60
                            
                            try {
                                dir(service.path) {
                                    // Check if Dockerfile exists
                                    if (!fileExists('Dockerfile')) {
                                        echo "⚠️ No Dockerfile found in ${service.path}, skipping Docker build..."
                                        buildResults[service.name] = 'SKIPPED - No Dockerfile'
                                        return
                                    }
                                    
                                    // Verify JAR file exists
                                    def jarExists = sh(
                                        script: 'ls target/*.jar 2>/dev/null | wc -l',
                                        returnStdout: true
                                    ).trim() != '0'
                                    
                                    if (!jarExists) {
                                        throw new Exception("No JAR file found in target/ directory")
                                    }
                                    
                                    def jarName = sh(
                                        script: 'basename $(ls target/*.jar | head -1)',
                                        returnStdout: true
                                    ).trim()
                                    
                                    echo "✅ Found JAR: ${jarName}"
                                    
                                    // Build Docker images
                                    def primaryImage = "${env.DOCKERHUB_USR}/${service.name}:${env.PRIMARY_TAG}"
                                    def secondaryImage = "${env.DOCKERHUB_USR}/${service.name}:${env.SECONDARY_TAG}"
                                    
                                    echo "🐳 Building Docker image..."
                                    sh "docker build -t ${primaryImage} ."
                                    
                                    // Tag with secondary tag if different
                                    if (env.PRIMARY_TAG != env.SECONDARY_TAG) {
                                        sh "docker tag ${primaryImage} ${secondaryImage}"
                                    }
                                    
                                    // Get image size
                                    def imageSize = sh(
                                        script: "docker images ${primaryImage} --format '{{.Size}}'",
                                        returnStdout: true
                                    ).trim()
                                    
                                    echo "✅ Image built successfully - Size: ${imageSize}"
                                    
                                    // Push images
                                    echo "📤 Pushing to Docker Hub..."
                                    sh "docker push ${primaryImage}"
                                    
                                    if (env.PRIMARY_TAG != env.SECONDARY_TAG) {
                                        sh "docker push ${secondaryImage}"
                                    }
                                    
                                    buildResults[service.name] = 'SUCCESS'
                                    successCount++
                                    
                                    echo "✅ Successfully pushed ${service.name}"
                                    echo "   🏷️ Tags: ${env.PRIMARY_TAG}, ${env.SECONDARY_TAG}"
                                    echo "   📦 Repository: https://hub.docker.com/r/${env.DOCKERHUB_USR}/${service.name}"
                                    
                                    // Clean up local image to save space
                                    sh "docker rmi ${primaryImage} || true"
                                    if (env.PRIMARY_TAG != env.SECONDARY_TAG) {
                                        sh "docker rmi ${secondaryImage} || true"
                                    }
                                }
                                
                            } catch (Exception e) {
                                buildResults[service.name] = "FAILED: ${e.getMessage()}"
                                echo "❌ Docker build failed for ${service.name}: ${e.getMessage()}"
                            }
                        } else {
                            echo "⏭️ Skipping ${service.name} (no changes detected)"
                        }
                    }
                    
                    // Store results for summary
                    env.DOCKER_BUILD_RESULTS = buildResults.collect { k, v -> "${k}:${v}" }.join('|')
                    env.DOCKER_SUCCESS_COUNT = successCount.toString()
                    env.DOCKER_TOTAL_COUNT = services.findAll { it.changed == "true" }.size().toString()
                }
            }
        }
        
        stage('📊 Build Summary') {
            steps {
                script {
                    echo ""
                    echo "=" * 70
                    echo "🎉 CI PIPELINE SUMMARY"
                    echo "=" * 70
                    echo "📍 Branch: ${env.BRANCH_NAME}"
                    echo "🔖 Commit: ${env.COMMIT_ID}"
                    echo "🏷️ Docker Tags: ${env.PRIMARY_TAG}, ${env.SECONDARY_TAG}"
                    echo "📊 Docker Success Rate: ${env.DOCKER_SUCCESS_COUNT}/${env.DOCKER_TOTAL_COUNT} services"
                    echo ""
                    
                    if (env.DOCKER_BUILD_RESULTS) {
                        def dockerResults = env.DOCKER_BUILD_RESULTS.split('\\|')
                        def successfulImages = []
                        def failedImages = []
                        
                        echo "🐳 Docker Build Results:"
                        dockerResults.each { result ->
                            def parts = result.split(':')
                            def service = parts[0]
                            def status = parts[1]
                            
                            if (status == 'SUCCESS') {
                                successfulImages.add(service)
                                echo "✅ ${service}"
                            } else {
                                failedImages.add(service)
                                echo "❌ ${service} - ${status}"
                            }
                        }
                        
                        if (successfulImages.size() > 0) {
                            echo ""
                            echo "🎯 Successfully built and pushed images:"
                            successfulImages.each { service ->
                                echo "   docker pull ${env.DOCKERHUB_USR}/${service}:${env.PRIMARY_TAG}"
                            }
                            echo ""
                            echo "🌐 Docker Hub repositories:"
                            successfulImages.each { service ->
                                echo "   https://hub.docker.com/r/${env.DOCKERHUB_USR}/${service}"
                            }
                        }
                        
                        if (failedImages.size() > 0) {
                            echo ""
                            echo "⚠️ Failed Docker builds: ${failedImages.join(', ')}"
                        }
                    }
                    
                    echo ""
                    echo "🏁 Pipeline completed at: ${new Date()}"
                }
            }
        }

        stage('📦 Update Helm Charts repo') {
            when { expression { env.IS_TAG_BUILD } }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔄 Cloning spring-petclinic-config to update image tags → ${env.PRIMARY_TAG}"
                        sh '''
                            rm -rf spring-petclinic-config || true
                            git clone https://$GITHUB_TOKEN@github.com/Tondeptrai23/spring-petclinic-config.git
                            cd spring-petclinic-config
                            git config user.email "22120375@student.hcmus.edu.vn"
                            git config user.name  "Tondeptrai23"

                            # Update every 'tag:' entry in the staging values file
                            sed -i -E "s/tag: .*/tag: ${PRIMARY_TAG}/g" helm-charts/staging/values.yaml

                            if git diff --quiet; then
                                echo '⚠️  No tag changes detected – nothing to commit'
                            else
                                git add helm-charts/staging/values.yaml
                                git commit -m "chore(ci): bump image tags to ${PRIMARY_TAG}"
                                git push origin main
                            fi
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Cleanup Docker
                echo "🧹 Cleaning up Docker resources..."
                sh '''
                    docker logout || true
                    docker system prune -f || true
                '''
            }
        }
        
        success {
            echo '🎉 Build, Test, and Docker Push successful!'
        }
        
        failure {
            echo '❌ Build, Test, or Docker Push failed!'
        }
        
        unstable {
            echo '⚠️ Build completed with some test failures!'
        }
        
        cleanup {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
    }
}
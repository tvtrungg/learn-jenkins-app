pipeline {
	agent any

	environment {
		NETLIFY_AUTH_TOKEN = credentials('netlify-token-jenkins')
	}

	stages {
		stage('Build') {
		    agent {
		        docker {
		            image 'node:18-alpine'
		            reuseNode true
		        }
		    }
		    steps {
		        sh '''
		            ls -la
		            npm ci
		            npm run build
		            ls -la
		        '''
		    }
		}

			stage('Tests') {
					parallel {
							stage('Unit Tests') {
									agent {
											docker {
													image 'node:18-alpine'
													reuseNode true
											}
									}

									steps {
											sh '''
													npm test
											'''
									}

									post {
											always {
													junit(
															testResults: 'test-results/junit.xml',
															allowEmptyResults: true
													)
											}
									}
							}

							stage('E2E Tests') {
									agent {
											docker {
													image 'mcr.microsoft.com/playwright:v1.64.1-noble'
													reuseNode true
													args '-u root:root'
											}
									}

									steps {
											sh '''
													npm install serve
													node_modules/.bin/serve -s build &
													sleep 5
													PLAYWRIGHT_HTML_DO_NOT_INLINE_ASSETS=1 \
															npx playwright test --reporter=html
											'''
									}

									post {
											always {
													publishHTML(target: [
															allowMissing: false,
															alwaysLinkToLastBuild: false,
															keepAll: false,
															reportDir: 'playwright-report',
															reportFiles: 'index.html',
															reportName: 'Playwright HTML Report'
													])
											}
									}
							}
						}
			}

			stage('Deploy') {
			    agent {
			        docker {
			            image 'node:18-alpine'
			            reuseNode true
			        }
			    }
			    steps {
			        sh '''
						echo "Deploying to Netlify"
			            npm install netlify-cli@20.1.1
						node_modules/.bin/netlify --version
						node_modules/.bin/netlify status
						node_modules/.bin/netlify deploy --dir=build --prod
			        '''
			    }
			}
		}
}

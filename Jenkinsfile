pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = 'cb8518d4-1d9f-44a4-8bd8-63fac9c056b1'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
  }

  stages {
    /*
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
          node --version
          npm --version
          npm ci
          npm run build
          ls -la
        '''
      }
    }
    */
    stage(Tests) {
      parallel {
        stage("Unit tests") {
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
          steps {
            sh '''
              #test -f build/index.html
              npm test
            '''
          }
          post {
            always {
              junit 'jest-results/junit.xml'
            }
          }
        }
        stage('E2E') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
              reuseNode true
            }
          }
          steps {
            sh '''
              npm install serve
              node_modules/.bin/serve -s build &
              sleep 10
              npx playwright test --reporter=html
            '''
          }
          post {
            always {
              publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
            }
          }
        }
      }
    }
    stage('Deploy staging') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          npm install netlify-cli
          node_modules/.bin/netlify --version
          echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy --dir=build
        '''
      }
    }
    stage('Deploy prod') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          npm install netlify-cli
          node_modules/.bin/netlify --version
          echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy --dir=build --prod
        '''
      }
    }
    stage('Prod E2E') {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
          reuseNode true
        }
      }
      environment {
        CI_ENVIRONMENT_URL = 'https://endearing-kitsune-b495aa.netlify.app/'
      }
      steps {
        sh '''
            npx playwright test  --reporter=html
        '''
      }
      post {
        always {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
    }
  }
}
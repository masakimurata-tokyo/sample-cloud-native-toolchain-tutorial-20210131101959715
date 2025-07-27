# Usage Examples and Quick Start Guide

## Table of Contents
1. [Quick Start](#quick-start)
2. [API Usage Examples](#api-usage-examples)
3. [Frontend Integration Examples](#frontend-integration-examples)
4. [Testing Examples](#testing-examples)
5. [Development Workflow Examples](#development-workflow-examples)
6. [Deployment Examples](#deployment-examples)
7. [Customization Examples](#customization-examples)

## Quick Start

### 1. Installation and Setup

```bash
# Clone or navigate to the project
cd devops-tutorial

# Install dependencies
npm install

# Start the application
npm start

# Application will be available at http://localhost:3000
```

### 2. Verify Installation

```bash
# Test the hello endpoint
curl http://localhost:3000/hello
# Expected: "Hello, stranger!"

# Test applications endpoint
curl http://localhost:3000/applications
# Expected: JSON array of applications

# Access Swagger UI
# Open browser: http://localhost:3000/docs
```

### 3. Development Setup

```bash
# Run linting
npm run lint

# Run tests
npm test

# Watch mode for development
gulp watch  # or grunt watch
```

## API Usage Examples

### Hello World Endpoint Examples

#### Basic Usage
```bash
# Simple greeting
curl -X GET "http://localhost:3000/hello"
# Response: "Hello, stranger!"

# Personalized greeting
curl -X GET "http://localhost:3000/hello?name=Developer"
# Response: "Hello, Developer!"

# URL-encoded names
curl -X GET "http://localhost:3000/hello?name=John%20Doe"
# Response: "Hello, John Doe!"
```

#### JavaScript/Node.js Examples
```javascript
// Using fetch API (modern browsers/Node.js with node-fetch)
async function sayHello(name = 'stranger') {
  const response = await fetch(`http://localhost:3000/hello?name=${encodeURIComponent(name)}`);
  const greeting = await response.json();
  console.log(greeting);
  return greeting;
}

// Usage
sayHello('Alice');  // "Hello, Alice!"
sayHello();         // "Hello, stranger!"

// Using Node.js http module
const http = require('http');

function getHello(name, callback) {
  const url = `http://localhost:3000/hello?name=${encodeURIComponent(name)}`;
  
  http.get(url, (res) => {
    let data = '';
    res.on('data', chunk => data += chunk);
    res.on('end', () => {
      callback(null, JSON.parse(data));
    });
  }).on('error', callback);
}

// Usage
getHello('Bob', (err, greeting) => {
  if (err) throw err;
  console.log(greeting); // "Hello, Bob!"
});
```

#### Python Examples
```python
import requests
import urllib.parse

def say_hello(name='stranger'):
    url = f"http://localhost:3000/hello?name={urllib.parse.quote(name)}"
    response = requests.get(url)
    return response.json()

# Usage
print(say_hello('Python'))    # "Hello, Python!"
print(say_hello())            # "Hello, stranger!"

# Async version with aiohttp
import aiohttp
import asyncio

async def say_hello_async(name='stranger'):
    url = f"http://localhost:3000/hello?name={urllib.parse.quote(name)}"
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# Usage
asyncio.run(say_hello_async('AsyncPython'))
```

### Applications Endpoint Examples

#### Get All Applications
```bash
# Basic request
curl -X GET "http://localhost:3000/applications" \
  -H "Accept: application/json"

# With pretty printing
curl -X GET "http://localhost:3000/applications" \
  -H "Accept: application/json" | jq '.'

# Save to file
curl -X GET "http://localhost:3000/applications" \
  -H "Accept: application/json" \
  -o applications.json
```

#### Get Specific Application
```bash
# Get application by ID
curl -X GET "http://localhost:3000/applications/1234" \
  -H "Accept: application/json"

# Test with different IDs
for id in 1234 1235 1236 9999; do
  echo "Testing ID: $id"
  curl -s "http://localhost:3000/applications/$id" | jq '.'
done
```

#### JavaScript Client Examples
```javascript
// Complete API client class
class DevOpsAPIClient {
  constructor(baseURL = 'http://localhost:3000') {
    this.baseURL = baseURL;
  }

  async makeRequest(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const response = await fetch(url, {
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  async getApplications() {
    return this.makeRequest('/applications');
  }

  async getApplication(id) {
    return this.makeRequest(`/applications/${id}`);
  }

  async sayHello(name = 'stranger') {
    return this.makeRequest(`/hello?name=${encodeURIComponent(name)}`);
  }

  // Utility methods
  async getOnlineApplications() {
    const apps = await this.getApplications();
    return apps.filter(app => app.status === 'online');
  }

  async getApplicationsByStatus(status) {
    const apps = await this.getApplications();
    return apps.filter(app => app.status === status);
  }

  async searchApplications(searchTerm) {
    const apps = await this.getApplications();
    return apps.filter(app => 
      app.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}

// Usage examples
const client = new DevOpsAPIClient();

// Basic usage
try {
  const apps = await client.getApplications();
  console.log(`Found ${apps.length} applications`);
  
  const firstApp = await client.getApplication(apps[0].id);
  console.log(`First app: ${firstApp.name}`);
  
  const greeting = await client.sayHello('API Client');
  console.log(greeting);
} catch (error) {
  console.error('API Error:', error.message);
}

// Advanced usage
async function analyzeApplications() {
  const client = new DevOpsAPIClient();
  
  try {
    const allApps = await client.getApplications();
    const onlineApps = await client.getOnlineApplications();
    const offlineApps = await client.getApplicationsByStatus('offline');
    
    console.log('Application Analysis:');
    console.log(`Total: ${allApps.length}`);
    console.log(`Online: ${onlineApps.length}`);
    console.log(`Offline: ${offlineApps.length}`);
    
    // Search for specific applications
    const toolchainApps = await client.searchApplications('toolchain');
    console.log(`Toolchain apps: ${toolchainApps.length}`);
    
    return {
      total: allApps.length,
      online: onlineApps.length,
      offline: offlineApps.length,
      toolchains: toolchainApps.length
    };
  } catch (error) {
    console.error('Analysis failed:', error);
    throw error;
  }
}

// Run analysis
analyzeApplications().then(stats => {
  console.log('Analysis complete:', stats);
});
```

## Frontend Integration Examples

### React Component Examples

```jsx
// ApplicationList component
import React, { useState, useEffect } from 'react';

function ApplicationList() {
  const [applications, setApplications] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchApplications() {
      try {
        const response = await fetch('/applications');
        if (!response.ok) throw new Error('Failed to fetch');
        const apps = await response.json();
        setApplications(apps);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchApplications();
  }, []);

  if (loading) return <div>Loading applications...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="application-list">
      <h2>Applications ({applications.length})</h2>
      <div className="app-grid">
        {applications.map(app => (
          <ApplicationCard key={app.id} application={app} />
        ))}
      </div>
    </div>
  );
}

function ApplicationCard({ application }) {
  const statusColors = {
    online: '#28a745',
    offline: '#dc3545',
    unknown: '#6c757d'
  };

  return (
    <div className="app-card">
      <h3>{application.name}</h3>
      <p className="app-id">ID: {application.id}</p>
      <p className="app-updated">
        Updated: {new Date(application.lastUpdated).toLocaleDateString()}
      </p>
      <span 
        className="status-badge"
        style={{ backgroundColor: statusColors[application.status] }}
      >
        {application.status}
      </span>
    </div>
  );
}

// HelloWorld component
function HelloWorld() {
  const [name, setName] = useState('');
  const [greeting, setGreeting] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await fetch(`/hello?name=${encodeURIComponent(name)}`);
      const result = await response.json();
      setGreeting(result);
    } catch (error) {
      console.error('Failed to get greeting:', error);
    }
  };

  return (
    <div className="hello-world">
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="Enter your name"
        />
        <button type="submit">Say Hello</button>
      </form>
      {greeting && <p className="greeting">{greeting}</p>}
    </div>
  );
}

export { ApplicationList, HelloWorld };
```

### Vue.js Component Examples

```vue
<!-- ApplicationDashboard.vue -->
<template>
  <div class="dashboard">
    <h1>DevOps Dashboard</h1>
    
    <div class="stats">
      <div class="stat-card">
        <h3>Total Apps</h3>
        <p>{{ applications.length }}</p>
      </div>
      <div class="stat-card">
        <h3>Online</h3>
        <p>{{ onlineCount }}</p>
      </div>
      <div class="stat-card">
        <h3>Offline</h3>
        <p>{{ offlineCount }}</p>
      </div>
    </div>

    <div class="applications">
      <h2>Applications</h2>
      <div class="filter">
        <select v-model="statusFilter" @change="filterApplications">
          <option value="">All Statuses</option>
          <option value="online">Online</option>
          <option value="offline">Offline</option>
          <option value="unknown">Unknown</option>
        </select>
      </div>
      
      <div class="app-list">
        <div 
          v-for="app in filteredApplications" 
          :key="app.id"
          class="app-item"
          :class="app.status"
        >
          <h4>{{ app.name }}</h4>
          <p>Last Updated: {{ formatDate(app.lastUpdated) }}</p>
          <span class="status">{{ app.status }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ApplicationDashboard',
  data() {
    return {
      applications: [],
      statusFilter: '',
      loading: true
    }
  },
  computed: {
    filteredApplications() {
      if (!this.statusFilter) return this.applications;
      return this.applications.filter(app => app.status === this.statusFilter);
    },
    onlineCount() {
      return this.applications.filter(app => app.status === 'online').length;
    },
    offlineCount() {
      return this.applications.filter(app => app.status === 'offline').length;
    }
  },
  async mounted() {
    await this.fetchApplications();
  },
  methods: {
    async fetchApplications() {
      try {
        const response = await fetch('/applications');
        this.applications = await response.json();
      } catch (error) {
        console.error('Failed to fetch applications:', error);
      } finally {
        this.loading = false;
      }
    },
    formatDate(dateString) {
      return new Date(dateString).toLocaleDateString();
    }
  }
}
</script>
```

### jQuery Examples

```javascript
// jQuery-based application manager
$(document).ready(function() {
  const AppManager = {
    baseURL: window.location.origin,
    
    init() {
      this.loadApplications();
      this.setupEventHandlers();
    },
    
    async loadApplications() {
      try {
        $('#loading').show();
        const response = await $.get(`${this.baseURL}/applications`);
        this.renderApplications(response);
      } catch (error) {
        this.showError('Failed to load applications');
      } finally {
        $('#loading').hide();
      }
    },
    
    renderApplications(apps) {
      const container = $('#applications');
      container.empty();
      
      apps.forEach(app => {
        const appElement = $(`
          <div class="app-card" data-id="${app.id}">
            <h3>${app.name}</h3>
            <p>Status: <span class="status ${app.status}">${app.status}</span></p>
            <p>Last Updated: ${new Date(app.lastUpdated).toLocaleDateString()}</p>
            <button class="btn-details" data-id="${app.id}">View Details</button>
          </div>
        `);
        container.append(appElement);
      });
    },
    
    setupEventHandlers() {
      // Handle application details
      $(document).on('click', '.btn-details', async (e) => {
        const appId = $(e.target).data('id');
        await this.showApplicationDetails(appId);
      });
      
      // Handle hello form
      $('#hello-form').on('submit', async (e) => {
        e.preventDefault();
        const name = $('#name-input').val();
        await this.sayHello(name);
      });
      
      // Status filter
      $('#status-filter').on('change', (e) => {
        this.filterByStatus($(e.target).val());
      });
    },
    
    async showApplicationDetails(appId) {
      try {
        const app = await $.get(`${this.baseURL}/applications/${appId}`);
        if (app.id) {
          this.showModal(`
            <h3>${app.name}</h3>
            <p><strong>ID:</strong> ${app.id}</p>
            <p><strong>Status:</strong> ${app.status}</p>
            <p><strong>Last Updated:</strong> ${app.lastUpdated}</p>
          `);
        } else {
          this.showError('Application not found');
        }
      } catch (error) {
        this.showError('Failed to load application details');
      }
    },
    
    async sayHello(name) {
      try {
        const greeting = await $.get(`${this.baseURL}/hello`, { name });
        $('#greeting-result').text(greeting).show();
      } catch (error) {
        this.showError('Failed to get greeting');
      }
    },
    
    filterByStatus(status) {
      if (!status) {
        $('.app-card').show();
        return;
      }
      
      $('.app-card').each(function() {
        const appStatus = $(this).find('.status').text();
        $(this).toggle(appStatus === status);
      });
    },
    
    showModal(content) {
      const modal = $(`
        <div class="modal">
          <div class="modal-content">
            <span class="close">&times;</span>
            ${content}
          </div>
        </div>
      `);
      
      $('body').append(modal);
      modal.show();
      
      modal.find('.close').on('click', () => modal.remove());
    },
    
    showError(message) {
      $('#error-message').text(message).show().delay(3000).fadeOut();
    }
  };
  
  AppManager.init();
});
```

## Testing Examples

### Unit Test Examples

```javascript
// test/unit/models/application.test.js
const { expect } = require('chai');
const applicationModel = require('../../../api/models/application');

describe('Application Model', () => {
  describe('getAllApplications', () => {
    it('should return an array of applications', () => {
      const apps = applicationModel.getAllApplications();
      expect(apps).to.be.an('array');
      expect(apps).to.have.length.greaterThan(0);
    });
    
    it('should return applications with required properties', () => {
      const apps = applicationModel.getAllApplications();
      apps.forEach(app => {
        expect(app).to.have.property('id');
        expect(app).to.have.property('name');
        expect(app).to.have.property('lastUpdated');
        expect(app).to.have.property('status');
      });
    });
  });
  
  describe('getApplicationById', () => {
    it('should return correct application for valid ID', () => {
      const app = applicationModel.getApplicationById(1234);
      expect(app).to.have.property('id', 1234);
      expect(app).to.have.property('name', 'Open Toolchain');
    });
    
    it('should return empty object for invalid ID', () => {
      const app = applicationModel.getApplicationById(9999);
      expect(app).to.deep.equal({});
    });
    
    it('should handle string IDs', () => {
      const app = applicationModel.getApplicationById('1234');
      expect(app).to.have.property('id', 1234);
    });
  });
});
```

### API Integration Tests

```javascript
// test/api/hello.test.js
const request = require('supertest');
const app = require('../../app');

describe('Hello API', () => {
  describe('GET /hello', () => {
    it('should return default greeting', (done) => {
      request(app)
        .get('/hello')
        .expect(200)
        .expect('Content-Type', /json/)
        .expect('"Hello, stranger!"')
        .end(done);
    });
    
    it('should return personalized greeting', (done) => {
      request(app)
        .get('/hello?name=John')
        .expect(200)
        .expect('"Hello, John!"')
        .end(done);
    });
    
    it('should handle URL-encoded names', (done) => {
      request(app)
        .get('/hello?name=John%20Doe')
        .expect(200)
        .expect('"Hello, John Doe!"')
        .end(done);
    });
    
    it('should handle empty name parameter', (done) => {
      request(app)
        .get('/hello?name=')
        .expect(200)
        .expect('"Hello, stranger!"')
        .end(done);
    });
  });
});

// test/api/applications.test.js
describe('Applications API', () => {
  describe('GET /applications', () => {
    it('should return array of applications', (done) => {
      request(app)
        .get('/applications')
        .expect(200)
        .expect('Content-Type', /json/)
        .expect((res) => {
          expect(res.body).to.be.an('array');
          expect(res.body).to.have.length(6);
        })
        .end(done);
    });
    
    it('should return applications with correct structure', (done) => {
      request(app)
        .get('/applications')
        .expect(200)
        .expect((res) => {
          res.body.forEach(app => {
            expect(app).to.have.all.keys('id', 'name', 'lastUpdated', 'status');
          });
        })
        .end(done);
    });
  });
  
  describe('GET /applications/:id', () => {
    it('should return specific application', (done) => {
      request(app)
        .get('/applications/1234')
        .expect(200)
        .expect((res) => {
          expect(res.body).to.have.property('id', 1234);
          expect(res.body).to.have.property('name', 'Open Toolchain');
        })
        .end(done);
    });
    
    it('should return empty object for invalid ID', (done) => {
      request(app)
        .get('/applications/9999')
        .expect(200)
        .expect((res) => {
          expect(res.body).to.deep.equal({});
        })
        .end(done);
    });
  });
});
```

### Load Testing Examples

```javascript
// test/load/load-test.js
const http = require('http');
const { performance } = require('perf_hooks');

class LoadTester {
  constructor(baseURL = 'http://localhost:3000') {
    this.baseURL = baseURL;
  }
  
  async runTest(endpoint, concurrency = 10, duration = 30000) {
    console.log(`Starting load test: ${endpoint}`);
    console.log(`Concurrency: ${concurrency}, Duration: ${duration}ms`);
    
    const results = {
      requests: 0,
      errors: 0,
      startTime: performance.now(),
      responseTimes: []
    };
    
    const workers = [];
    for (let i = 0; i < concurrency; i++) {
      workers.push(this.worker(endpoint, duration, results));
    }
    
    await Promise.all(workers);
    
    const endTime = performance.now();
    const totalTime = endTime - results.startTime;
    
    return this.generateReport(results, totalTime);
  }
  
  async worker(endpoint, duration, results) {
    const endTime = Date.now() + duration;
    
    while (Date.now() < endTime) {
      const startTime = performance.now();
      
      try {
        await this.makeRequest(endpoint);
        results.requests++;
        results.responseTimes.push(performance.now() - startTime);
      } catch (error) {
        results.errors++;
      }
    }
  }
  
  makeRequest(endpoint) {
    return new Promise((resolve, reject) => {
      const url = `${this.baseURL}${endpoint}`;
      http.get(url, (res) => {
        let data = '';
        res.on('data', chunk => data += chunk);
        res.on('end', () => resolve(data));
      }).on('error', reject);
    });
  }
  
  generateReport(results, totalTime) {
    const responseTimes = results.responseTimes;
    responseTimes.sort((a, b) => a - b);
    
    return {
      totalRequests: results.requests,
      totalErrors: results.errors,
      errorRate: (results.errors / (results.requests + results.errors) * 100).toFixed(2),
      requestsPerSecond: (results.requests / (totalTime / 1000)).toFixed(2),
      averageResponseTime: (responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length).toFixed(2),
      minResponseTime: responseTimes[0]?.toFixed(2) || 0,
      maxResponseTime: responseTimes[responseTimes.length - 1]?.toFixed(2) || 0,
      p95ResponseTime: responseTimes[Math.floor(responseTimes.length * 0.95)]?.toFixed(2) || 0
    };
  }
}

// Usage
async function runLoadTests() {
  const tester = new LoadTester();
  
  console.log('Testing Hello endpoint...');
  const helloResults = await tester.runTest('/hello?name=LoadTest', 5, 10000);
  console.log(helloResults);
  
  console.log('\nTesting Applications endpoint...');
  const appsResults = await tester.runTest('/applications', 10, 15000);
  console.log(appsResults);
}

// Run if called directly
if (require.main === module) {
  runLoadTests().catch(console.error);
}

module.exports = LoadTester;
```

## Development Workflow Examples

### Automated Development Setup

```bash
#!/bin/bash
# scripts/dev-setup.sh

echo "Setting up development environment..."

# Install dependencies
npm install

# Setup git hooks
echo "Setting up git hooks..."
cp scripts/pre-commit .git/hooks/
chmod +x .git/hooks/pre-commit

# Create local environment file
if [ ! -f .env ]; then
  echo "Creating .env file..."
  cat > .env << EOF
NODE_ENV=development
PORT=3000
DEBUG=app:*
NEW_RELIC_LICENSE_KEY=
EOF
fi

# Start development services
echo "Starting development services..."
npm run dev &

echo "Development environment ready!"
echo "Application: http://localhost:3000"
echo "API Docs: http://localhost:3000/docs"
```

### Git Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running pre-commit checks..."

# Run linting
echo "Linting code..."
npm run lint
if [ $? -ne 0 ]; then
  echo "Linting failed. Please fix errors before committing."
  exit 1
fi

# Run tests
echo "Running tests..."
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Please fix tests before committing."
  exit 1
fi

echo "Pre-commit checks passed!"
```

### Docker Development

```dockerfile
# Dockerfile.dev
FROM node:16-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Development command with nodemon
CMD ["npm", "run", "dev"]
```

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev
```

## Deployment Examples

### Production Deployment

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

echo "Starting deployment process..."

# Build application
echo "Building application..."
npm run build

# Run tests
echo "Running tests..."
npm test

# Deploy to staging
echo "Deploying to staging..."
cf push devops-tutorial-staging -f manifest-staging.yml

# Run smoke tests
echo "Running smoke tests..."
npm run smoke-test -- --url https://devops-tutorial-staging.mybluemix.net

# Deploy to production
echo "Deploying to production..."
cf push devops-tutorial -f manifest.yml

echo "Deployment complete!"
```

### Cloud Foundry Manifest

```yaml
# manifest.yml
applications:
- name: devops-tutorial
  memory: 256M
  instances: 2
  buildpack: sdk-for-nodejs
  command: npm start
  env:
    NODE_ENV: production
    NEW_RELIC_LICENSE_KEY: ((new-relic-key))
  services:
    - devops-tutorial-db
```

### Kubernetes Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-tutorial
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devops-tutorial
  template:
    metadata:
      labels:
        app: devops-tutorial
    spec:
      containers:
      - name: app
        image: devops-tutorial:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"

---
apiVersion: v1
kind: Service
metadata:
  name: devops-tutorial-service
spec:
  selector:
    app: devops-tutorial
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

## Customization Examples

### Adding New API Endpoints

```javascript
// api/controllers/health.js
'use strict';

module.exports = {
  health: health,
  metrics: metrics
};

function health(req, res) {
  const healthStatus = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    version: require('../../package.json').version
  };
  
  res.json(healthStatus);
}

function metrics(req, res) {
  const metrics = {
    requests: global.requestCount || 0,
    errors: global.errorCount || 0,
    responseTime: global.averageResponseTime || 0,
    timestamp: new Date().toISOString()
  };
  
  res.json(metrics);
}
```

```yaml
# Update api/swagger/swagger.yaml
paths:
  /health:
    x-swagger-router-controller: health
    get:
      description: Returns health status
      operationId: health
      responses:
        "200":
          description: Success
          schema:
            type: object
            properties:
              status:
                type: string
              timestamp:
                type: string
              uptime:
                type: number
              memory:
                type: object
              version:
                type: string
```

### Custom Middleware

```javascript
// middleware/requestLogger.js
module.exports = function requestLogger() {
  return function(req, res, next) {
    const startTime = Date.now();
    
    // Increment request counter
    global.requestCount = (global.requestCount || 0) + 1;
    
    // Log request
    console.log(`${req.method} ${req.path} - ${req.ip}`);
    
    // Capture response time
    res.on('finish', () => {
      const responseTime = Date.now() - startTime;
      console.log(`${req.method} ${req.path} - ${res.statusCode} - ${responseTime}ms`);
      
      // Update average response time
      global.averageResponseTime = global.averageResponseTime 
        ? (global.averageResponseTime + responseTime) / 2 
        : responseTime;
    });
    
    next();
  };
};

// Use in app.js
const requestLogger = require('./middleware/requestLogger');
app.use(requestLogger());
```

### Custom Templates

```handlebars
{{!-- views/dashboard.hbs --}}
<div class="dashboard">
  <h1>DevOps Dashboard</h1>
  
  <div class="metrics">
    <div class="metric-card">
      <h3>Total Applications</h3>
      <span class="metric-value">{{totalApps}}</span>
    </div>
    <div class="metric-card">
      <h3>Online Applications</h3>
      <span class="metric-value">{{onlineApps}}</span>
    </div>
    <div class="metric-card">
      <h3>System Health</h3>
      <span class="metric-value {{healthStatus}}">{{healthStatus}}</span>
    </div>
  </div>
  
  <div class="recent-activity">
    <h2>Recent Activity</h2>
    <ul>
      {{#each activities}}
      <li>
        <span class="timestamp">{{timestamp}}</span>
        <span class="activity">{{description}}</span>
      </li>
      {{/each}}
    </ul>
  </div>
</div>
```

This comprehensive usage guide provides practical examples for every aspect of the DevOps Tutorial application, from basic API usage to advanced deployment scenarios. Each example includes complete, working code that can be used as a starting point for development and customization.

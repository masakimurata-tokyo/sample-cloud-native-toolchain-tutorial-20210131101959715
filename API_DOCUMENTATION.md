# DevOps Tutorial App - API Documentation

## Table of Contents
1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [API Endpoints](#api-endpoints)
4. [Models](#models)
5. [Controllers](#controllers)
6. [Frontend Components](#frontend-components)
7. [Configuration](#configuration)
8. [Error Handling](#error-handling)
9. [Testing](#testing)
10. [Examples](#examples)

## Overview

The DevOps Tutorial App is a Node.js Express application that demonstrates a simple REST API with Swagger documentation. It provides endpoints for managing applications and includes a basic web interface.

### Key Features
- RESTful API with Swagger documentation
- Application management endpoints
- Handlebars templating for web views
- Express.js framework
- Built-in Swagger UI
- Comprehensive testing suite

### Technology Stack
- **Backend**: Node.js, Express.js
- **API Documentation**: Swagger/OpenAPI 2.0
- **Template Engine**: Handlebars (HBS)
- **Testing**: Mocha, Karma, Supertest
- **Build Tools**: Grunt, Gulp
- **Monitoring**: New Relic (optional)

## Getting Started

### Prerequisites
- Node.js >= 0.10.0
- npm

### Installation
```bash
npm install
```

### Running the Application
```bash
npm start
# or
node app.js
```

The application will start on port 3000 by default (or PORT environment variable).

### Swagger UI
Access the interactive API documentation at: `http://localhost:3000/docs`

## API Endpoints

### Base URL
- **Development**: `http://localhost:3000`
- **Production**: Configure via environment

### Content Types
- **Request**: `application/json`
- **Response**: `application/json`, `text/plain`, `text/html`

---

### 1. Hello World Endpoint

#### GET /hello
Simple greeting endpoint that returns a personalized hello message.

**Parameters:**
- `name` (query, optional): Name to include in greeting
  - Type: `string`
  - Default: `"stranger"`

**Responses:**
- `200`: Success
  - Content-Type: `application/json`
  - Body: `string` - Greeting message

**Example Request:**
```bash
curl "http://localhost:3000/hello?name=John"
```

**Example Response:**
```json
"Hello, John!"
```

**Without name parameter:**
```bash
curl "http://localhost:3000/hello"
```
```json
"Hello, stranger!"
```

---

### 2. Applications Endpoints

#### GET /applications
Retrieves a list of all applications.

**Parameters:** None

**Responses:**
- `200`: Success
  - Content-Type: `application/json`
  - Body: `Array<Application>` - Array of application objects

**Example Request:**
```bash
curl "http://localhost:3000/applications"
```

**Example Response:**
```json
[
  {
    "id": 1234,
    "name": "Open Toolchain",
    "lastUpdated": "2015-07-10T14:48:00",
    "status": "online"
  },
  {
    "id": 1235,
    "name": "BizDevOps",
    "lastUpdated": "2015-07-10T17:48:00",
    "status": "online"
  }
]
```

#### GET /applications/{appId}
Retrieves a specific application by ID.

**Parameters:**
- `appId` (path, required): Application ID
  - Type: `string`

**Responses:**
- `200`: Success
  - Content-Type: `application/json`
  - Body: `Application` - Application object
- `404`: Application not found (returns empty object)

**Example Request:**
```bash
curl "http://localhost:3000/applications/1234"
```

**Example Response:**
```json
{
  "id": 1234,
  "name": "Open Toolchain",
  "lastUpdated": "2015-07-10T14:48:00",
  "status": "online"
}
```

---

### 3. Web Interface Endpoints

#### GET /
Renders the main application page with API documentation and links.

**Parameters:** None

**Responses:**
- `200`: HTML page with application interface

**Features:**
- Links to all available API endpoints
- Link to Swagger UI documentation
- IBM Cloud DevOps Services branding

## Models

### Application Model
The Application model represents an application entity in the system.

**Location**: `api/models/application.js`

#### Properties
```javascript
{
  id: number,           // Unique identifier
  name: string,         // Application name
  lastUpdated: string,  // ISO 8601 timestamp
  status: string        // Status: 'online', 'offline', 'unknown'
}
```

#### Methods

##### `getApplicationById(appId)`
Retrieves a single application by ID.

**Parameters:**
- `appId` (string|number): Application ID to search for

**Returns:**
- `Object`: Application object if found, empty object if not found

**Example:**
```javascript
const applicationModel = require('./api/models/application');
const app = applicationModel.getApplicationById(1234);
console.log(app.name); // "Open Toolchain"
```

##### `getAllApplications()`
Retrieves all applications.

**Parameters:** None

**Returns:**
- `Array<Object>`: Array of all application objects

**Example:**
```javascript
const applicationModel = require('./api/models/application');
const apps = applicationModel.getAllApplications();
console.log(apps.length); // 6
```

#### Sample Data
The model includes 6 predefined applications:
1. **Open Toolchain** (ID: 1234, Status: online)
2. **BizDevOps** (ID: 1235, Status: online)
3. **Deployment Advisor** (ID: 1236, Status: offline)
4. **Topology Analytics** (ID: 1237, Status: offline)
5. **Timeline Analytics** (ID: 1238, Status: online)
6. **Configuration Toolchain** (ID: 1239, Status: unknown)

## Controllers

### Hello World Controller
**Location**: `api/controllers/hello_world.js`

#### `hello(req, res)`
Handles the hello world endpoint.

**Parameters:**
- `req` (Object): Express request object
  - `req.swagger.params.name.value`: Name parameter from query string
- `res` (Object): Express response object

**Functionality:**
- Extracts name parameter from request
- Defaults to "stranger" if no name provided
- Returns formatted greeting message as JSON

**Example Usage:**
```javascript
// Called automatically by Swagger-Express middleware
// when GET /hello is requested
```

### Applications Controller
**Location**: `api/controllers/applications.js`

#### `applications(req, res)`
Handles both list and single application endpoints.

**Parameters:**
- `req` (Object): Express request object
  - `req.swagger.params.appId.value`: Application ID (if provided)
- `res` (Object): Express response object

**Functionality:**
- If `appId` parameter exists: returns single application
- If no `appId` parameter: returns all applications
- Uses application model for data retrieval

**Example Usage:**
```javascript
// Called automatically by Swagger-Express middleware
// for both /applications and /applications/{appId} routes
```

## Frontend Components

### Templates

#### Layout Template
**Location**: `views/layout.hbs`

Base template that provides the HTML structure for all pages.

**Features:**
- Responsive viewport meta tag
- CSS normalization and custom styles
- Clean HTML5 structure

**Variables:**
- `title`: Page title

#### Index Template
**Location**: `views/index.hbs`

Main application page template.

**Features:**
- IBM Cloud DevOps Services header
- API endpoint documentation
- Interactive links to test endpoints
- Link to Swagger UI

**Variables:**
- `title`: Application title

#### Error Templates
- `views/error.hbs`: Error page template
- `views/404.hbs`: 404 not found page

### Client-Side JavaScript

#### Console Plugins
**Location**: `public/javascripts/plugins.js`

Provides console method stubs for browsers that lack a console.

**Features:**
- Prevents console errors in production
- Stubs all standard console methods
- Safe fallback for debugging calls

**Methods Stubbed:**
- `assert`, `clear`, `count`, `debug`, `dir`, `dirxml`
- `error`, `exception`, `group`, `groupCollapsed`, `groupEnd`
- `info`, `log`, `markTimeline`, `profile`, `profileEnd`
- `table`, `time`, `timeEnd`, `timeline`, `timelineEnd`
- `timeStamp`, `trace`, `warn`

### Stylesheets

#### Main Styles
**Location**: `public/stylesheets/main.css`

Custom application styles providing:
- IBM Cloud branding
- Responsive layout
- Button and link styling
- Content area formatting

#### Normalize CSS
**Location**: `public/stylesheets/normalize.css`

CSS reset/normalization for cross-browser consistency.

## Configuration

### Application Configuration
**Location**: `config/default.yaml`

**Settings:**
```yaml
swagger:
  mapErrorsToJson: true
  docEndpoints:
    raw: /swagger
```

### Environment Variables
- `PORT`: Server port (default: 3000)
- `NEW_RELIC_LICENSE_KEY`: New Relic monitoring (optional)
- `NODE_ENV`: Environment mode (development/production)

### Package Configuration
**Location**: `package.json`

**Key Scripts:**
- `npm start`: Start the application
- `npm test`: Run test suite
- `npm run lint`: Run code linting

## Error Handling

### Error Response Format
All API errors return a standardized error response:

```json
{
  "message": "Error description"
}
```

### Error Types

#### Development Mode
- Full error stack traces included
- Detailed error information

#### Production Mode
- Minimal error information
- No stack traces leaked to users

### HTTP Status Codes
- `200`: Success
- `404`: Not found
- `500`: Internal server error

## Testing

### Test Structure
**Location**: `test/`

#### Unit Tests
- **API Tests**: `test/api/`
- **Unit Tests**: `test/unit/`

#### Test Scripts
```bash
# Run all tests
npm test

# Run unit tests only
npm run mocha_unit_test

# Run API tests
npm run mocha_unit_test_api

# Run functional tests
npm run mocha_fvt_test
```

#### Testing Tools
- **Mocha**: Test framework
- **Karma**: Browser testing
- **Supertest**: HTTP assertion testing
- **Chai**: Assertion library
- **Sinon**: Mocking library

### Example Test Structure
```javascript
// Example API test
const request = require('supertest');
const app = require('../app');

describe('GET /hello', function() {
  it('should return hello message', function(done) {
    request(app)
      .get('/hello?name=test')
      .expect(200)
      .expect('"Hello, test!"')
      .end(done);
  });
});
```

## Examples

### Complete API Usage Examples

#### 1. Testing the Hello Endpoint
```bash
# Basic hello
curl -X GET "http://localhost:3000/hello"
# Response: "Hello, stranger!"

# Personalized hello
curl -X GET "http://localhost:3000/hello?name=Developer"
# Response: "Hello, Developer!"
```

#### 2. Working with Applications
```bash
# Get all applications
curl -X GET "http://localhost:3000/applications" \
  -H "Accept: application/json"

# Get specific application
curl -X GET "http://localhost:3000/applications/1234" \
  -H "Accept: application/json"
```

#### 3. Using the Application Programmatically
```javascript
const express = require('express');
const app = require('./app'); // Import the app

// Example: Custom middleware to log API calls
app.use('/api/*', (req, res, next) => {
  console.log(`API Call: ${req.method} ${req.path}`);
  next();
});

// Example: Adding custom routes
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});
```

#### 4. Extending the Application Model
```javascript
// api/models/application.js - Example extension
function addApplication(appData) {
  const newId = Math.max(...APPS.map(app => app.id)) + 1;
  const newApp = {
    id: newId,
    name: appData.name,
    lastUpdated: new Date().toISOString(),
    status: appData.status || 'unknown'
  };
  APPS.push(newApp);
  return newApp;
}

function updateApplicationStatus(appId, status) {
  const app = getApplicationById(appId);
  if (app.id) {
    app.status = status;
    app.lastUpdated = new Date().toISOString();
  }
  return app;
}

module.exports = {
  getApplicationById,
  getAllApplications,
  addApplication,
  updateApplicationStatus
};
```

### Integration Examples

#### 1. Using with Frontend Frameworks
```javascript
// Example: Fetch applications in a React component
useEffect(() => {
  fetch('/applications')
    .then(response => response.json())
    .then(apps => setApplications(apps))
    .catch(error => console.error('Error:', error));
}, []);
```

#### 2. API Client Example
```javascript
class DevOpsAPIClient {
  constructor(baseURL = 'http://localhost:3000') {
    this.baseURL = baseURL;
  }
  
  async getApplications() {
    const response = await fetch(`${this.baseURL}/applications`);
    return response.json();
  }
  
  async getApplication(id) {
    const response = await fetch(`${this.baseURL}/applications/${id}`);
    return response.json();
  }
  
  async sayHello(name = 'stranger') {
    const response = await fetch(`${this.baseURL}/hello?name=${encodeURIComponent(name)}`);
    return response.json();
  }
}

// Usage
const client = new DevOpsAPIClient();
const apps = await client.getApplications();
console.log(`Found ${apps.length} applications`);
```

---

## Troubleshooting

### Common Issues

1. **Port already in use**
   ```bash
   # Change port
   PORT=3001 npm start
   ```

2. **Missing dependencies**
   ```bash
   npm install
   ```

3. **Swagger UI not loading**
   - Check that swagger.yaml is valid
   - Verify swagger-express-mw configuration

### Debug Mode
```bash
DEBUG=* npm start
```

### Logs
- Application logs are output to console
- Use environment variable `NODE_ENV=development` for verbose logging

---

## Contributing

1. Follow the existing code style
2. Add tests for new functionality
3. Update documentation for API changes
4. Run `npm run lint` before submitting changes

## License

MIT License - see LICENSE file for details.
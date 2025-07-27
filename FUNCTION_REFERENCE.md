# Function Reference Guide

## Table of Contents
1. [API Controllers](#api-controllers)
2. [Model Functions](#model-functions)
3. [Utility Functions](#utility-functions)
4. [Configuration Functions](#configuration-functions)
5. [Error Handlers](#error-handlers)
6. [Middleware Functions](#middleware-functions)

## API Controllers

### Hello World Controller (`api/controllers/hello_world.js`)

#### `hello(req, res)`
Handles the greeting endpoint functionality.

**Type**: Express route handler

**Parameters**:
- `req` (Object): Express request object
  - `req.swagger.params.name` (Object): Swagger parameter object
    - `req.swagger.params.name.value` (string): The name to greet
- `res` (Object): Express response object

**Returns**: void (sends JSON response)

**Behavior**:
1. Extracts the `name` parameter from the Swagger request object
2. Defaults to `'stranger'` if no name is provided
3. Formats a greeting string using `util.format()`
4. Sends the greeting as a JSON response

**Example Usage**:
```javascript
// This function is called automatically by Swagger middleware
// when a GET request is made to /hello

// Internal flow:
// 1. Request: GET /hello?name=John
// 2. Swagger extracts 'John' into req.swagger.params.name.value
// 3. Function formats: "Hello, John!"
// 4. Response: JSON string "Hello, John!"
```

**Dependencies**:
- `util` module for string formatting

**Error Handling**:
- No explicit error handling; relies on Express error middleware
- Malformed requests handled by Swagger validation

---

### Applications Controller (`api/controllers/applications.js`)

#### `applications(req, res)`
Handles both listing all applications and retrieving a specific application.

**Type**: Express route handler

**Parameters**:
- `req` (Object): Express request object
  - `req.swagger.params.appId` (Object, optional): Swagger parameter for application ID
    - `req.swagger.params.appId.value` (string): The application ID to retrieve
- `res` (Object): Express response object

**Returns**: void (sends JSON response)

**Behavior**:
1. Checks if `appId` parameter exists in the request
2. If `appId` exists:
   - Calls `applicationModel.getApplicationById(appId)`
   - Returns single application object
3. If no `appId`:
   - Calls `applicationModel.getAllApplications()`
   - Returns array of all applications

**Example Usage**:
```javascript
// For GET /applications (list all)
// req.swagger.params.appId is undefined
// Returns: Array of all applications

// For GET /applications/1234 (get specific)
// req.swagger.params.appId.value = "1234"
// Returns: Single application object or empty object
```

**Dependencies**:
- `applicationModel` from `../models/application`

**Error Handling**:
- No explicit error handling
- Invalid appId returns empty object (handled by model)

## Model Functions

### Application Model (`api/models/application.js`)

#### `getApplicationById(appId)`
Retrieves a single application by its ID.

**Type**: Pure function

**Parameters**:
- `appId` (string|number): The application ID to search for

**Returns**:
- `Object`: Application object if found
- `Object`: Empty object `{}` if not found

**Behavior**:
1. Iterates through the `APPS` array
2. Compares each application's `id` with the provided `appId`
3. Uses loose equality (`==`) for comparison
4. Returns the first matching application
5. Returns empty object if no match found

**Example Usage**:
```javascript
const applicationModel = require('./api/models/application');

// Valid ID
const app = applicationModel.getApplicationById(1234);
console.log(app); 
// Output: { id: 1234, name: "Open Toolchain", lastUpdated: "2015-07-10T14:48:00", status: "online" }

// Invalid ID
const notFound = applicationModel.getApplicationById(9999);
console.log(notFound); 
// Output: {}

// String ID (auto-converted)
const app2 = applicationModel.getApplicationById("1234");
console.log(app2.name); 
// Output: "Open Toolchain"
```

**Time Complexity**: O(n) where n is the number of applications

**Space Complexity**: O(1)

#### `getAllApplications()`
Retrieves all applications in the system.

**Type**: Pure function

**Parameters**: None

**Returns**:
- `Array<Object>`: Array containing all application objects

**Behavior**:
1. Returns the entire `APPS` array
2. No filtering or transformation applied
3. Returns reference to original array (not a copy)

**Example Usage**:
```javascript
const applicationModel = require('./api/models/application');

const allApps = applicationModel.getAllApplications();
console.log(allApps.length); // Output: 6
console.log(allApps[0].name); // Output: "Open Toolchain"

// Iterate through all applications
allApps.forEach(app => {
  console.log(`${app.name}: ${app.status}`);
});
```

**Time Complexity**: O(1)

**Space Complexity**: O(1) - returns reference, not copy

**Data Structure**:
```javascript
// Each application object has the structure:
{
  id: number,           // Unique identifier
  name: string,         // Application name
  lastUpdated: string,  // ISO 8601 timestamp
  status: string        // 'online', 'offline', or 'unknown'
}
```

## Utility Functions

### Console Plugins (`public/javascripts/plugins.js`)

#### Anonymous IIFE (Immediately Invoked Function Expression)
Provides safe console method stubs for browsers lacking console support.

**Type**: IIFE utility function

**Parameters**: None

**Returns**: void (modifies global console object)

**Behavior**:
1. Defines a no-operation function (`noop`)
2. Lists all standard console methods
3. Checks if each method exists on the console object
4. Stubs missing methods with the no-op function

**Example Usage**:
```javascript
// This runs automatically when the script loads
// After execution, all console methods are safe to use:

console.log("This won't error even in old browsers");
console.warn("Neither will this");
console.debug("Or this");
```

**Methods Protected**:
```javascript
const methods = [
  'assert', 'clear', 'count', 'debug', 'dir', 'dirxml', 'error',
  'exception', 'group', 'groupCollapsed', 'groupEnd', 'info', 'log',
  'markTimeline', 'profile', 'profileEnd', 'table', 'time', 'timeEnd',
  'timeline', 'timelineEnd', 'timeStamp', 'trace', 'warn'
];
```

**Browser Compatibility**: 
- Ensures console methods work in IE < 9
- Prevents runtime errors in production environments
- Safe fallback for debugging calls

## Configuration Functions

### Swagger Configuration
The application uses Swagger Express middleware for API management.

#### Swagger Setup (`app.js`)

**Configuration Object**:
```javascript
var config = {
  appRoot: __dirname, // required config
};
```

**Initialization Function**:
```javascript
SwaggerExpress.create(config, function(err, swaggerExpress) {
  if (err) {
    throw err;
  }

  // add swagger-ui
  app.use(new SwaggerUi(swaggerExpress.runner.swagger));

  // install middleware
  swaggerExpress.register(app);

  var port = process.env.PORT || 3000;
  app.listen(port);
});
```

**Parameters**:
- `config` (Object): Swagger configuration with `appRoot` property
- `callback` (Function): Callback with `err` and `swaggerExpress` parameters

**Behavior**:
1. Creates Swagger Express instance
2. Adds Swagger UI middleware at `/docs`
3. Registers API routes from `swagger.yaml`
4. Starts the server on specified port

## Error Handlers

### Development Error Handler
Provides detailed error information in development mode.

**Type**: Express error middleware

**Parameters**:
- `err` (Error): Error object
- `req` (Object): Express request object
- `res` (Object): Express response object
- `next` (Function): Next middleware function

**Behavior**:
1. Sets status code from error or defaults to 500
2. Renders error template with full error details
3. Includes stack trace in development mode

**Example Usage**:
```javascript
// Automatically called when errors occur in development
// Only active when app.get('env') === 'development'

app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: err, // Full error object with stack trace
  });
});
```

### Production Error Handler
Provides minimal error information in production mode.

**Type**: Express error middleware

**Parameters**:
- `err` (Error): Error object
- `req` (Object): Express request object
- `res` (Object): Express response object
- `next` (Function): Next middleware function

**Behavior**:
1. Sets status code from error or defaults to 500
2. Renders error template with minimal details
3. Hides stack traces and sensitive information

**Example Usage**:
```javascript
// Default error handler for production
// No stack traces leaked to users

app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}, // Empty object - no sensitive data
  });
});
```

## Middleware Functions

### Express Standard Middleware
The application uses several Express middleware functions:

#### Body Parser Middleware
```javascript
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```

**Purpose**: Parses incoming request bodies
- `bodyParser.json()`: Parses JSON payloads
- `bodyParser.urlencoded()`: Parses URL-encoded form data

#### Static File Middleware
```javascript
app.use(express.static(path.join(__dirname, 'public')));
```

**Purpose**: Serves static files from the `public` directory
**Behavior**: 
- Maps `/` to `public/` directory
- Serves CSS, JavaScript, images, and other static assets

#### Cookie Parser Middleware
```javascript
app.use(cookieParser());
```

**Purpose**: Parses cookies from request headers
**Behavior**: Populates `req.cookies` object

#### Morgan Logger Middleware
```javascript
app.use(logger('dev'));
```

**Purpose**: HTTP request logging
**Format**: Development format showing method, URL, status, response time

### Route Middleware

#### Home Route Handler (`routes/index.js`)
```javascript
router.get('/', function(req, res) {
  res.render('index', { title: 'DevOps Tutorial App' });
});
```

**Type**: Express route handler

**Parameters**:
- `req` (Object): Express request object
- `res` (Object): Express response object

**Behavior**:
1. Renders the `index` Handlebars template
2. Passes `title` variable to template
3. Returns HTML response

**Template Variables**:
- `title`: "DevOps Tutorial App"

---

## Function Usage Patterns

### Error Handling Pattern
Most functions in the application follow these error handling patterns:

1. **Controller Functions**: Rely on Express error middleware
2. **Model Functions**: Return empty objects/arrays for not-found cases
3. **Middleware Functions**: Pass errors to next middleware via `next(err)`

### Callback Pattern
The application uses traditional callback patterns for asynchronous operations:

```javascript
// Swagger initialization
SwaggerExpress.create(config, function(err, swaggerExpress) {
  if (err) throw err;
  // Continue setup
});
```

### Module Export Pattern
All modules follow CommonJS export patterns:

```javascript
// Single function export
module.exports = {
  functionName: functionName,
  anotherFunction: anotherFunction
};

// Direct assignment
module.exports = app;
```

### Request Parameter Pattern
API controllers access parameters through Swagger request object:

```javascript
// Query parameters
const name = req.swagger.params.name.value;

// Path parameters  
const appId = req.swagger.params.appId.value;
```

This ensures type validation and automatic parameter extraction according to the Swagger specification.
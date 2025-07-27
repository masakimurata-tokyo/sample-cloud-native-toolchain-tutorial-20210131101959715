# Component Guide

## Table of Contents
1. [Template Components](#template-components)
2. [Stylesheet Components](#stylesheet-components)
3. [JavaScript Components](#javascript-components)
4. [Static Asset Components](#static-asset-components)
5. [Build and Configuration Components](#build-and-configuration-components)
6. [Component Integration](#component-integration)

## Template Components

### Layout Template (`views/layout.hbs`)

The base layout template that provides the HTML structure for all pages.

#### Structure
```handlebars
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{title}}</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel='stylesheet' href='/stylesheets/normalize.css' />
    <link rel='stylesheet' href='/stylesheets/main.css' />
  </head>
  <body>
    {{{body}}}
  </body>
</html>
```

#### Features
- **Responsive Design**: Includes viewport meta tag for mobile optimization
- **CSS Normalization**: Links to normalize.css for cross-browser consistency
- **Custom Styling**: Links to main.css for application-specific styles
- **Template Variables**: Supports `{{title}}` variable injection
- **Body Content**: Uses `{{{body}}}` for unescaped HTML content injection

#### Usage
```javascript
// In Express controller
res.render('templateName', { 
  title: 'Page Title'
});
```

#### Dependencies
- `/stylesheets/normalize.css` - CSS reset/normalization
- `/stylesheets/main.css` - Application styles

---

### Index Template (`views/index.hbs`)

The main application landing page template.

#### Structure
```handlebars
<div class="header">
  <div class="left">
    <a class="button-link" href="https://cloud.ibm.com/devops">
      <img class="bluemix-logo" src="..." />
      <span class="devopsservices-text">IBM <span class="bluemix-text">Cloud</span> DevOps Services</span>
    </a>
  </div>
  <div class="right">
    <span class="app-title">{{title}}</span>
  </div>
</div>
<div class="content">
  <!-- API documentation links -->
</div>
```

#### Features
- **IBM Cloud Branding**: Header with IBM Cloud DevOps Services logo and branding
- **API Documentation**: Interactive links to test API endpoints
- **Swagger Integration**: Link to `/docs` for Swagger UI
- **Responsive Layout**: Flexible header and content sections

#### API Links Provided
1. **Hello Endpoint**: `/hello?name=joe` - Demonstrates query parameters
2. **Applications List**: `/applications` - Shows all applications
3. **Specific Application**: `/applications/1234` - Shows path parameters
4. **Swagger Documentation**: `/docs` - Interactive API explorer

#### Template Variables
- `{{title}}`: Application title (passed from controller)

#### CSS Classes Used
- `.header` - Main header container
- `.left` / `.right` - Header layout sections
- `.button-link` - IBM Cloud branding link
- `.bluemix-logo` - Logo image styling
- `.devopsservices-text` / `.bluemix-text` - Text branding
- `.app-title` - Application title styling
- `.content` - Main content area
- `.description` - API endpoint descriptions

---

### Error Template (`views/error.hbs`)

Simple error display template.

#### Structure
```handlebars
<h1>{{message}}</h1>
<h2>{{error.status}}</h2>
<pre>{{error.stack}}</pre>
```

#### Features
- **Error Message Display**: Shows the error message
- **Status Code**: Displays HTTP status code
- **Stack Trace**: Shows stack trace (development mode only)

#### Usage Context
- Automatically rendered by Express error handlers
- Development vs. production error information varies

---

### 404 Template (`views/404.hbs`)

Comprehensive 404 not found page.

#### Features
- **Error Information**: Displays 404 status and message
- **Navigation Help**: Provides links back to main application
- **Consistent Styling**: Matches application design

## Stylesheet Components

### Main Stylesheet (`public/stylesheets/main.css`)

Application-specific styles providing IBM Cloud branding and responsive design.

#### Key Style Categories

##### Header Styles
```css
.header {
  /* Header container styling */
}

.left, .right {
  /* Header layout sections */
}

.button-link {
  /* IBM Cloud branding link */
}

.bluemix-logo {
  /* Logo image styling */
}
```

##### Typography
```css
.devopsservices-text {
  /* Service name styling */
}

.bluemix-text {
  /* IBM Cloud brand text */
}

.app-title {
  /* Application title styling */
}
```

##### Layout
```css
.content {
  /* Main content area */
}

.description {
  /* API description text */
}
```

#### Features
- **IBM Cloud Branding**: Official colors and typography
- **Responsive Design**: Flexible layouts for different screen sizes
- **Clean Typography**: Readable fonts and appropriate sizing
- **Interactive Elements**: Hover states and link styling

#### Color Scheme
- Primary brand colors from IBM Cloud palette
- Consistent with IBM Design Language
- Accessible contrast ratios

#### Responsive Breakpoints
- Mobile-first approach
- Tablet and desktop optimizations
- Flexible grid system

---

### Normalize CSS (`public/stylesheets/normalize.css`)

CSS reset and normalization library for cross-browser consistency.

#### Purpose
- **Browser Reset**: Removes default browser styling inconsistencies
- **Baseline Styles**: Provides consistent starting point
- **Bug Fixes**: Addresses known browser-specific CSS issues

#### Coverage
- Typography normalization
- Form element consistency
- HTML5 element support
- Mobile webkit fixes

#### Version
Based on normalize.css v8.0.1 (latest stable version)

## JavaScript Components

### Console Safety Plugin (`public/javascripts/plugins.js`)

Browser compatibility utility that prevents console-related errors.

#### Purpose
Ensures console methods are always available, preventing runtime errors in production environments or older browsers.

#### Implementation
```javascript
(function() {
  var method;
  var noop = function() {};
  var methods = [
    'assert', 'clear', 'count', 'debug', 'dir', 'dirxml', 'error',
    'exception', 'group', 'groupCollapsed', 'groupEnd', 'info', 'log',
    'markTimeline', 'profile', 'profileEnd', 'table', 'time', 'timeEnd',
    'timeline', 'timelineEnd', 'timeStamp', 'trace', 'warn'
  ];
  var length = methods.length;

  while (length--) {
    method = methods[length];
    if (!console[method]) {
      console[method] = noop;
    }
  }
}());
```

#### Features
- **IIFE Pattern**: Self-executing function to avoid global pollution
- **No-op Fallbacks**: Provides safe no-operation functions
- **Complete Coverage**: Stubs all standard console methods
- **Production Safety**: Prevents console errors in deployed applications

#### Browser Compatibility
- **IE < 9**: Provides console object when missing
- **Mobile Browsers**: Handles webkit-specific console issues
- **Debug Mode**: Safe for development with debugging tools

---

### Main JavaScript (`public/javascripts/main.js`)

Currently empty placeholder for application-specific JavaScript.

#### Intended Usage
- Custom application logic
- API interaction code
- UI enhancement scripts
- Event handlers

#### Integration Points
```javascript
// Example usage patterns:

// API calls
fetch('/applications')
  .then(response => response.json())
  .then(data => console.log(data));

// UI interactions
document.addEventListener('DOMContentLoaded', function() {
  // Initialize application
});

// Event handling
document.querySelectorAll('.api-link').forEach(link => {
  link.addEventListener('click', handleAPICall);
});
```

## Static Asset Components

### Robots.txt (`public/robots.txt`)

Search engine crawler configuration.

#### Content
```
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /

Sitemap: /sitemap.xml
```

#### Purpose
- **SEO Control**: Guides search engine indexing
- **Security**: Prevents indexing of sensitive areas
- **Performance**: Reduces unnecessary crawler traffic

---

### Humans.txt (`public/humans.txt`)

Human-readable information about the development team and technology stack.

#### Content
```
/* TEAM */
Developer: IBM DevOps Team
Contact: devops [at] ibm.com
Location: Austin, TX, USA

/* THANKS */
Node.js Community
Express.js Team
Swagger Community

/* SITE */
Last update: 2023
Language: English
Doctype: HTML5
Standards: HTML5, CSS3, ES2015+
Components: Node.js, Express, Swagger, Handlebars
```

#### Purpose
- **Team Recognition**: Credits development team
- **Technology Stack**: Documents tools and frameworks used
- **Community Thanks**: Acknowledges open source contributions

---

### Browser Configuration (`public/browserconfig.xml`)

Microsoft browser configuration for Windows 8/10 tile customization.

#### Features
- **Live Tiles**: Configures Windows Start menu tiles
- **Brand Colors**: Sets tile background colors
- **Icon Specifications**: Defines tile icon images

---

### Cross-Domain Policy (`public/crossdomain.xml`)

Adobe Flash cross-domain policy configuration.

#### Purpose
- **Legacy Support**: Supports older Flash-based integrations
- **Security**: Controls cross-domain access permissions
- **API Access**: Allows controlled external API access

## Build and Configuration Components

### Grunt Configuration (`gruntfile.js`)

Build automation and task management configuration.

#### Available Tasks
```javascript
// Example task registration
grunt.registerTask('test', ['jshint', 'mochaTest']);
grunt.registerTask('lint', ['jshint']);
grunt.registerTask('build', ['clean', 'concat', 'uglify']);
```

#### Features
- **Code Quality**: JSHint linting configuration
- **Testing**: Mocha test runner integration
- **File Watching**: Automatic rebuilds on file changes
- **Concurrent Tasks**: Parallel task execution

#### Task Categories
1. **Linting**: Code quality checks
2. **Testing**: Unit and integration tests
3. **Building**: Asset compilation and minification
4. **Deployment**: Production build preparation

---

### Gulp Configuration (`gulpfile.js`)

Modern build system configuration with streaming builds.

#### Task Examples
```javascript
gulp.task('lint', function() {
  return gulp.src(['**/*.js', '!node_modules/**'])
    .pipe(jshint())
    .pipe(jshint.reporter('default'));
});

gulp.task('test', function() {
  return gulp.src(['test/**/*.js'])
    .pipe(mocha({reporter: 'spec'}));
});
```

#### Features
- **Streaming Builds**: Efficient file processing
- **Watch Mode**: Automatic task execution on file changes
- **Plugin System**: Extensible task capabilities
- **Parallel Execution**: Concurrent task processing

#### Task Categories
1. **Development**: Live reload and watching
2. **Testing**: Karma and Mocha integration
3. **Production**: Minification and optimization
4. **Deployment**: Build artifact generation

---

### Package Configuration (`package.json`)

Node.js project configuration and dependency management.

#### Key Sections
```json
{
  "scripts": {
    "start": "node app.js",
    "test": "grunt test:sauce:parallel",
    "lint": "gulp lint"
  },
  "dependencies": {
    "express": "^4.17.1",
    "swagger-express-mw": "^0.7.0"
  },
  "devDependencies": {
    "mocha": "^6.1.4",
    "grunt": "~1.0.4"
  }
}
```

#### Features
- **Script Automation**: npm script definitions
- **Dependency Management**: Production and development dependencies
- **Version Control**: Semantic versioning specification
- **Metadata**: Project description and repository information

## Component Integration

### Template Rendering Flow

1. **Request Received**: Express routes handle incoming requests
2. **Controller Logic**: Route handlers process business logic
3. **Template Selection**: Appropriate template chosen (index, error, etc.)
4. **Variable Injection**: Data passed to template context
5. **Layout Rendering**: Base layout.hbs provides HTML structure
6. **Content Insertion**: Specific template content inserted into `{{{body}}}`
7. **Asset Loading**: CSS and JavaScript files loaded
8. **Response Sent**: Complete HTML delivered to client

### Asset Pipeline

1. **Static Files**: Express serves files from `/public` directory
2. **CSS Loading**: normalize.css → main.css loading order
3. **JavaScript Loading**: plugins.js → main.js execution order
4. **Caching**: Browser caching headers for static assets
5. **Compression**: Gzip compression for text assets

### Development Workflow

1. **File Watching**: Gulp/Grunt watch for file changes
2. **Automatic Linting**: Code quality checks on save
3. **Live Reload**: Browser refresh on template/style changes
4. **Test Execution**: Automatic test runs on code changes
5. **Error Reporting**: Console and browser error display

### Production Optimization

1. **Asset Minification**: CSS and JavaScript compression
2. **Cache Headers**: Long-term caching for static assets
3. **Error Handling**: Production-safe error pages
4. **Performance Monitoring**: New Relic integration
5. **Security Headers**: XSS and CSRF protection

---

## Component Extension Guide

### Adding New Templates

1. **Create Template File**: Add `.hbs` file to `/views` directory
2. **Define Route**: Create Express route to render template
3. **Add Styles**: Include template-specific CSS in `main.css`
4. **Test Rendering**: Verify template renders correctly
5. **Update Documentation**: Document new template component

### Adding New Stylesheets

1. **Create CSS File**: Add file to `/public/stylesheets`
2. **Update Layout**: Include stylesheet link in `layout.hbs`
3. **Define Variables**: Use CSS custom properties for theming
4. **Test Responsiveness**: Verify across device sizes
5. **Optimize Performance**: Minimize and compress for production

### Adding New JavaScript Components

1. **Create JS File**: Add file to `/public/javascripts`
2. **Update Layout**: Include script tag in `layout.hbs` or specific templates
3. **Handle Dependencies**: Ensure proper loading order
4. **Add Error Handling**: Graceful degradation for script failures
5. **Test Compatibility**: Verify across browser versions

This component guide provides comprehensive documentation for all frontend and build components in the DevOps Tutorial application. Each component is documented with its purpose, structure, features, and integration points to help developers understand and extend the application effectively.
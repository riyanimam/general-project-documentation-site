# Automated GitHub to New Relic Synthetics

[![JavaScript](https://img.shields.io/badge/JavaScript-ES6-F7DF1E?logo=javascript)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=github-actions)](https://github.com/features/actions)
[![New Relic](https://img.shields.io/badge/Platform-New%20Relic-00AC69?logo=newrelic)](https://newrelic.com/)

Automates syncing and managing New Relic Synthetics monitors from changes in a GitHub
repository (e.g., using GitHub Actions to push synthetic script updates or trigger
monitor updates). This empowers your CI/CD pipeline to keep synthetics in sync with
source control and enforce observability workflows.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/automated-github-to-new-relic-synthetics){ .md-button }

## Features

- **GitOps for Synthetics**: Manage synthetic monitors as code
- **GitHub Actions Integration**: Automated deployment on push
- **Script Management**: Version control for synthetic scripts
- **Monitor Configuration**: Declarative monitor definitions
- **Multi-Environment**: Support for dev, staging, production
- **Validation**: Pre-deployment script validation
- **Rollback Support**: Easy rollback to previous versions

## Prerequisites

- **Node.js** >= 18
- **New Relic Account** with Synthetics capability
- **New Relic API Key** (User API key with appropriate permissions)
- **GitHub Repository** for storing synthetic scripts

## Quick Start

```bash
# Clone the repository
git clone https://github.com/riyanimam/automated-github-to-new-relic-synthetics.git
cd automated-github-to-new-relic-synthetics

# Install dependencies
npm install

# Configure New Relic credentials
export NEW_RELIC_API_KEY="your-api-key"
export NEW_RELIC_ACCOUNT_ID="your-account-id"

# Deploy synthetics
npm run deploy
```

## Project Structure

```text
automated-github-to-new-relic-synthetics/
├── synthetics/
│   ├── monitors/
│   │   ├── website-health.yml       # Monitor config
│   │   ├── api-endpoint.yml         # API monitor
│   │   └── user-flow.yml            # Scripted browser
│   ├── scripts/
│   │   ├── website-health.js        # Monitor script
│   │   ├── api-endpoint.js
│   │   └── user-flow.js
│   └── shared/
│       └── helpers.js               # Shared utilities
├── .github/
│   └── workflows/
│       ├── deploy-synthetics.yml    # Deployment workflow
│       └── validate-synthetics.yml  # Validation workflow
├── scripts/
│   ├── deploy.js                    # Deployment script
│   ├── validate.js                  # Validation script
│   └── rollback.js                  # Rollback script
├── package.json
└── README.md
```

## Monitor Configuration

### Simple Monitor Example

```yaml
# synthetics/monitors/website-health.yml
name: Website Health Check
type: SIMPLE
frequency: 5  # minutes
locations:
  - AWS_US_EAST_1
  - AWS_EU_WEST_1
status: ENABLED
uri: https://example.com
validation_string: "Welcome"
verify_ssl: true
```

### Scripted Browser Example

```yaml
# synthetics/monitors/user-flow.yml
name: User Login Flow
type: SCRIPT_BROWSER
frequency: 15
locations:
  - AWS_US_EAST_1
status: ENABLED
script: user-flow.js
runtime:
  runtimeType: CHROME_BROWSER
  runtimeTypeVersion: "100"
```

### API Test Example

```yaml
# synthetics/monitors/api-endpoint.yml
name: API Health Check
type: SCRIPT_API
frequency: 1  # Every minute
locations:
  - AWS_US_WEST_2
status: ENABLED
script: api-endpoint.js
```

## Synthetic Scripts

### Simple HTTP Check

```javascript
// synthetics/scripts/api-endpoint.js
const assert = require('assert');
const https = require('https');

const options = {
  hostname: 'api.example.com',
  port: 443,
  path: '/health',
  method: 'GET',
  headers: {
    'Content-Type': 'application/json'
  }
};

https.get(options, (res) => {
  assert.equal(res.statusCode, 200, 'Expected status code 200');

  let data = '';
  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('end', () => {
    const json = JSON.parse(data);
    assert.equal(json.status, 'healthy', 'Service should be healthy');
    console.log('Health check passed!');
  });
}).on('error', (err) => {
  console.error('Health check failed:', err.message);
  throw err;
});
```

### Scripted Browser Test

```javascript
// synthetics/scripts/user-flow.js
const assert = require('assert');

$browser.get('https://example.com/login').then(() => {
  // Wait for login form
  return $browser.waitForAndFindElement($driver.By.id('username'));
}).then((element) => {
  // Enter username
  return element.sendKeys($secure.USERNAME);
}).then(() => {
  // Enter password
  return $browser.findElement($driver.By.id('password'))
    .then(el => el.sendKeys($secure.PASSWORD));
}).then(() => {
  // Click login button
  return $browser.findElement($driver.By.css('button[type="submit"]'))
    .then(el => el.click());
}).then(() => {
  // Wait for dashboard
  return $browser.waitForAndFindElement(
    $driver.By.css('.dashboard'),
    10000
  );
}).then(() => {
  // Verify login success
  return $browser.getCurrentUrl();
}).then((url) => {
  assert(url.includes('/dashboard'), 'Should redirect to dashboard');
  console.log('Login flow completed successfully!');
});
```

### API Test with Authentication

```javascript
// synthetics/scripts/authenticated-api.js
const assert = require('assert');
const https = require('https');

// Get auth token
function getAuthToken() {
  return new Promise((resolve, reject) => {
    const authOptions = {
      hostname: 'api.example.com',
      path: '/auth/token',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      }
    };

    const req = https.request(authOptions, (res) => {
      let data = '';
      res.on('data', (chunk) => data += chunk);
      res.on('end', () => {
        const json = JSON.parse(data);
        resolve(json.token);
      });
    });

    req.write(JSON.stringify({
      username: $secure.API_USERNAME,
      password: $secure.API_PASSWORD
    }));
    req.end();
  });
}

// Test protected endpoint
getAuthToken().then((token) => {
  const options = {
    hostname: 'api.example.com',
    path: '/api/protected/resource',
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${token}`
    }
  };

  https.get(options, (res) => {
    assert.equal(res.statusCode, 200);
    console.log('Protected endpoint accessible!');
  });
});
```

## GitHub Actions Workflow

### Automated Deployment

```yaml
# .github/workflows/deploy-synthetics.yml
name: Deploy Synthetics

on:
  push:
    branches: [main]
    paths:
      - 'synthetics/**'
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Validate synthetic scripts
        run: npm run validate

      - name: Deploy to New Relic
        env:
          NEW_RELIC_API_KEY: ${{ secrets.NEW_RELIC_API_KEY }}
          NEW_RELIC_ACCOUNT_ID: ${{ secrets.NEW_RELIC_ACCOUNT_ID }}
        run: npm run deploy

      - name: Notify on failure
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Synthetics Deployment Failed',
              body: 'Check the workflow run for details.'
            })
```

### Validation Workflow

```yaml
# .github/workflows/validate-synthetics.yml
name: Validate Synthetics

on:
  pull_request:
    paths:
      - 'synthetics/**'

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Validate YAML configs
        run: npm run validate:yaml

      - name: Lint JavaScript
        run: npm run lint

      - name: Test scripts
        run: npm run test
```

## Deployment Script

```javascript
// scripts/deploy.js
const fs = require('fs').promises;
const path = require('path');
const axios = require('axios');
const yaml = require('js-yaml');

const NEW_RELIC_API = 'https://api.newrelic.com';
const API_KEY = process.env.NEW_RELIC_API_KEY;
const ACCOUNT_ID = process.env.NEW_RELIC_ACCOUNT_ID;

async function deployMonitor(configPath, scriptPath) {
  // Read monitor config
  const configFile = await fs.readFile(configPath, 'utf8');
  const config = yaml.load(configFile);

  // Read script if applicable
  let script = null;
  if (scriptPath) {
    script = await fs.readFile(scriptPath, 'utf8');
  }

  // Create or update monitor
  const payload = {
    name: config.name,
    type: config.type,
    frequency: config.frequency,
    locations: config.locations,
    status: config.status,
    uri: config.uri,
    script: script
  };

  try {
    const response = await axios.post(
      `${NEW_RELIC_API}/v3/monitors`,
      payload,
      {
        headers: {
          'Api-Key': API_KEY,
          'Content-Type': 'application/json'
        }
      }
    );

    console.log(`✓ Deployed monitor: ${config.name}`);
    return response.data;
  } catch (error) {
    console.error(`✗ Failed to deploy ${config.name}:`, error.message);
    throw error;
  }
}

async function deployAll() {
  const monitorsDir = path.join(__dirname, '../synthetics/monitors');
  const scriptsDir = path.join(__dirname, '../synthetics/scripts');

  const monitors = await fs.readdir(monitorsDir);

  for (const monitor of monitors) {
    if (!monitor.endsWith('.yml')) continue;

    const configPath = path.join(monitorsDir, monitor);
    const scriptName = monitor.replace('.yml', '.js');
    const scriptPath = path.join(scriptsDir, scriptName);

    let script = null;
    try {
      await fs.access(scriptPath);
      script = scriptPath;
    } catch {
      // No script file
    }

    await deployMonitor(configPath, script);
  }

  console.log('\n✓ All monitors deployed successfully!');
}

deployAll().catch((error) => {
  console.error('Deployment failed:', error);
  process.exit(1);
});
```

## Gotchas & Tips

!!! warning "API Key Security"
    Never commit API keys to your repository. Use GitHub Secrets for
    sensitive credentials.

!!! tip "Script Validation"
    Test synthetic scripts locally before deploying:
    ```bash
    node synthetics/scripts/your-script.js
    ```

!!! note "Location Selection"
    Choose locations closest to your users for accurate performance metrics.
    Available locations:
    - AWS_US_EAST_1, AWS_US_WEST_2
    - AWS_EU_WEST_1, AWS_AP_SOUTHEAST_1
    - And many more...

!!! warning "Frequency Limits"
    Synthetic monitors have frequency limits based on your New Relic plan.
    Too frequent checks can consume your monthly allowance quickly.

!!! tip "Secure Credentials"
    Use New Relic's Secure Credentials feature for sensitive data in scripts:
    ```javascript
    const username = $secure.USERNAME;
    const password = $secure.PASSWORD;
    ```

## Troubleshooting

### Deployment Fails

```bash
# Validate YAML syntax
npm run validate:yaml

# Check API key permissions
curl -H "Api-Key: $NEW_RELIC_API_KEY" \
  https://api.newrelic.com/v2/users.json

# Test script locally
node synthetics/scripts/your-script.js
```

### Monitor Not Running

1. Check monitor status in New Relic UI
2. Verify locations are valid
3. Check script for syntax errors
4. Review New Relic Synthetics logs

### Script Errors

```javascript
// Add detailed logging
console.log('Step 1: Loading page...');
$browser.get(url).then(() => {
  console.log('Step 2: Page loaded');
  // ... more steps with logging
});

// Handle errors gracefully
$browser.get(url)
  .then(() => /* success */)
  .catch((err) => {
    console.error('Error details:', err);
    throw err;
  });
```

## Enhancement Ideas

- [ ] Add support for alert conditions
- [ ] Implement canary deployments
- [ ] Add performance baseline tracking
- [ ] Create Terraform provider integration
- [ ] Add monitor dependency mapping
- [ ] Implement A/B testing for monitors
- [ ] Add Slack/Teams notifications
- [ ] Create dashboard for monitor health

## Resources

- [New Relic Synthetics Documentation](https://docs.newrelic.com/docs/synthetics/)
- [Synthetics API Reference](https://docs.newrelic.com/docs/apis/synthetics-rest-api/)
- [Scripted Browser Examples](https://docs.newrelic.com/docs/synthetics/synthetic-monitoring/scripting-monitors/introduction-scripted-browser-monitors/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

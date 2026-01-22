# Common Issues

Frequently encountered problems and their solutions across all projects.

## Installation Issues

### Node.js Version Mismatch

**Symptom:**
```
error engine node: wanted: ">=20.0.0" (current: "18.x.x")
```

**Solution:**
```bash
# Using nvm
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version
```

### Python Version Issues

**Symptom:**
```
Python 3.12 required, but Python 3.9 found
```

**Solution:**
```bash
# Using pyenv
pyenv install 3.12
pyenv global 3.12

# Or specify in shebang
#!/usr/bin/env python3.12
```

### pnpm Not Found

**Symptom:**
```
pnpm: command not found
```

**Solution:**
```bash
# Install pnpm
npm install -g pnpm

# Or via corepack (Node.js 16+)
corepack enable
corepack prepare pnpm@latest --activate
```

## Dependency Issues

### Package Lock Conflict

**Symptom:**
```
ERR_PNPM_OUTDATED_LOCKFILE
```

**Solution:**
```bash
# Regenerate lock file
rm pnpm-lock.yaml
pnpm install
```

### Python Dependency Conflicts

**Symptom:**
```
ERROR: Cannot install package-a and package-b because these package versions have conflicting dependencies.
```

**Solution:**
```bash
# Use virtual environment
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Or use uv for faster resolution
pip install uv
uv pip install -r requirements.txt --system
```

### Go Module Issues

**Symptom:**
```
go: module found but does not contain package
```

**Solution:**
```bash
# Clear and re-download
go clean -modcache
go mod download
go mod tidy
```

## Build Issues

### TypeScript Compilation Errors

**Symptom:**
```
error TS2307: Cannot find module './utils' or its corresponding type declarations.
```

**Solution:**
```bash
# Check tsconfig.json paths
# Ensure file exists and is properly exported

# Rebuild
rm -rf dist/
pnpm run build
```

### ESM Import Issues

**Symptom:**
```
Error [ERR_REQUIRE_ESM]: require() of ES Module not supported
```

**Solution:**

Ensure `package.json` has:
```json
{
  "type": "module"
}
```

And imports use `.js` extension:
```typescript
// Wrong
import { handler } from './handler';

// Correct
import { handler } from './handler.js';
```

### Go Build Failures

**Symptom:**
```
cannot find module providing package github.com/aws/aws-lambda-go
```

**Solution:**
```bash
# Initialize modules
go mod init

# Download dependencies
go mod download

# Verify
go build ./...
```

## Test Issues

### Tests Not Found

**Symptom:**
```
No tests found
```

**Solution:**

Check test file naming:
- **TypeScript**: `*.test.ts` or `*.spec.ts`
- **Python**: `test_*.py` or `*_test.py`
- **Go**: `*_test.go`

### Test Timeout

**Symptom:**
```
Timeout - Async callback was not invoked within timeout
```

**Solution:**

Increase timeout in test config:

=== "Vitest"

    ```typescript
    // vitest.config.ts
    export default {
      test: {
        testTimeout: 30000
      }
    };
    ```

=== "pytest"

    ```bash
    pytest --timeout=30
    ```

=== "Go"

    ```bash
    go test -timeout 30s ./...
    ```

### Mock Issues

**Symptom:**
```
TypeError: Cannot read property of undefined
```

**Solution:**

Ensure mocks are properly set up:
```typescript
// Vitest
vi.mock('aws-sdk', () => ({
  S3: vi.fn().mockImplementation(() => ({
    getObject: vi.fn()
  }))
}));
```

## AWS Issues

### Credentials Not Found

**Symptom:**
```
Unable to locate credentials
```

**Solution:**
```bash
# Configure credentials
aws configure

# Or set environment variables
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
export AWS_DEFAULT_REGION=us-east-1

# Verify
aws sts get-caller-identity
```

### Access Denied

**Symptom:**
```
An error occurred (AccessDenied) when calling the X operation
```

**Solution:**

1. Check IAM policy has required permissions
2. Verify resource ARN is correct
3. Check for SCP (Service Control Policy) restrictions
4. Ensure MFA is not required

```bash
# Debug IAM
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/myuser \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::mybucket/*
```

### Region Mismatch

**Symptom:**
```
Resource not found in region us-east-1
```

**Solution:**
```bash
# Set correct region
export AWS_DEFAULT_REGION=us-west-2

# Or in AWS SDK
import boto3
client = boto3.client('s3', region_name='us-west-2')
```

## Git Issues

### Merge Conflicts

**Symptom:**
```
CONFLICT (content): Merge conflict in file.ts
```

**Solution:**
```bash
# View conflicts
git status

# Open file and resolve conflicts
# Look for <<<<<<< HEAD, =======, >>>>>>>

# After resolving
git add <file>
git commit
```

### Detached HEAD

**Symptom:**
```
You are in 'detached HEAD' state.
```

**Solution:**
```bash
# Create branch from current state
git checkout -b my-new-branch

# Or return to a branch
git checkout main
```

### Permission Denied (SSH)

**Symptom:**
```
git@github.com: Permission denied (publickey).
```

**Solution:**
```bash
# Check SSH key
ssh -T git@github.com

# Add SSH key
ssh-add ~/.ssh/id_ed25519

# Or use HTTPS instead
git remote set-url origin https://github.com/user/repo.git
```

## IDE Issues

### VS Code TypeScript Issues

**Symptom:**
```
Cannot find module or type declarations
```

**Solution:**
1. Restart TypeScript server: `Cmd/Ctrl + Shift + P` → "Restart TS Server"
2. Reload window: `Cmd/Ctrl + Shift + P` → "Reload Window"
3. Delete `node_modules` and reinstall

### Python Interpreter Not Found

**Symptom:**
```
Python interpreter not found
```

**Solution:**
1. `Cmd/Ctrl + Shift + P` → "Python: Select Interpreter"
2. Choose the correct virtual environment
3. Ensure Python extension is installed

### Go Tools Not Working

**Symptom:**
```
gopls was not able to find modules in your workspace
```

**Solution:**
1. Ensure `go.mod` exists in workspace root
2. Run `Go: Install/Update Tools` from command palette
3. Check GOPATH and GOROOT settings

## Performance Issues

### Slow Builds

**Causes:**
- Too many files being processed
- No caching enabled
- Insufficient resources

**Solutions:**
```bash
# Use incremental builds
pnpm run build --incremental

# Enable caching
# In CI, cache node_modules, .cache directories

# Use faster tools
# Biome instead of ESLint + Prettier
# uv instead of pip
```

### Memory Issues

**Symptom:**
```
JavaScript heap out of memory
```

**Solution:**
```bash
# Increase Node.js memory
export NODE_OPTIONS="--max-old-space-size=4096"

# Or in package.json
"scripts": {
  "build": "NODE_OPTIONS='--max-old-space-size=4096' tsc"
}
```

## Environment-Specific Issues

### Windows Path Issues

**Symptom:**
```
Error: ENOENT: no such file or directory
```

**Solution:**
```bash
# Use forward slashes or path.join
const filePath = path.join(__dirname, 'file.txt');

# Normalize paths
const normalized = path.normalize(rawPath);
```

### Line Ending Issues

**Symptom:**
```
warning: LF will be replaced by CRLF
```

**Solution:**
```bash
# Configure Git
git config --global core.autocrlf input  # macOS/Linux
git config --global core.autocrlf true   # Windows

# Use .gitattributes
* text=auto
*.sh text eol=lf
*.bat text eol=crlf
```

## Still Stuck?

If you can't find a solution here:

1. Check project-specific troubleshooting:
   - [Lambda Debugging](lambda-debugging.md)
   - [Terraform Issues](terraform-issues.md)

2. Search GitHub Issues in the relevant repository

3. Open a new issue with detailed information:
   - Error message
   - Steps to reproduce
   - Environment details
   - What you've already tried

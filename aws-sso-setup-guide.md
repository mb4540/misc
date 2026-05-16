# AWS SSO Setup Guide for jemba9-dev

## Current Status
- ✅ Homebrew installed
- ✅ AWS CLI installed (version 2.34.48)
- 🔄 Currently configuring SSO profile

## Issue Encountered
The SSO start URL `https://jemba9.awsapps.com/start` returned an "Invalid start url" error.

## Next Steps - Complete SSO Configuration

You have an AWS configure SSO command running that's waiting for input. Please provide these values:

### Configuration Values to Enter:

1. **SSO session name (Recommended):** 
   ```
   jemba9-dev
   ```

2. **SSO start URL:**
   Try one of these formats:
   ```
   https://jemba9.awsapps.com/start#/
   ```
   OR
   ```
   https://jemba9.awsapps.com/start/
   ```
   OR (original)
   ```
   https://jemba9.awsapps.com/start
   ```

3. **SSO region:**
   ```
   us-east-1
   ```

4. **SSO registration scopes:**
   ```
   (Just press Enter for default: sso:account:access)
   ```

5. **CLI default client Region:**
   ```
   us-west-2
   ```

6. **CLI default output format:**
   ```
   json
   ```

7. **CLI profile name:**
   ```
   jemba9-dev
   ```

---

## After Configuration Completes

### Step 1: Log in to AWS SSO
```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
aws sso login --profile jemba9-dev
```
This will open a browser window - approve access with your Jemba9 credentials.

### Step 2: Set Profile for Current Terminal Session
```bash
export AWS_PROFILE=jemba9-dev
```

### Step 3: Add to ~/.zshrc for Permanent Setup
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
echo 'export AWS_PROFILE=jemba9-dev' >> ~/.zshrc
```

### Step 4: Verify Login is Working
```bash
aws sts get-caller-identity
```
Should show account **824999955649**

---

## Important Notes

- **Token expires after ~8 hours**
- When you see: "SSO session associated with this profile has expired"
- Re-run: `aws sso login --profile jemba9-dev`

---

## Account Details
- **Profile:** jemba9-dev
- **Account ID:** 824999955649
- **SSO Region:** us-east-1
- **Default Region:** us-west-2
- **Output Format:** json

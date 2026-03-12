---
name: pinme
version: 1.0.0
description: This skill should be used when the user asks to "deploy", "upload", "publish" a frontend project, static website, or build output. Also activates when user mentions "pinme", "IPFS deployment", or wants to host static files online.
author:
  name: glitternetwork
  url: https://github.com/glitternetwork
homepage: https://github.com/glitternetwork/skills/tree/main/pinme
repository: https://github.com/glitternetwork/skills
license: MIT
keywords:
  - deploy
  - ipfs
  - static-site
  - pinme
  - hosting
  - frontend
  - web3
---

# PinMe Skill

Use PinMe CLI to deploy static files to IPFS and get a preview URL.

## When to Use

- User requests deployment of a frontend project
- User wants to upload static files/website
- User mentions "pinme", "deploy", "publish", or "host"
- After building a frontend project (Vue, React, Next.js, etc.)

## Deployment Steps

### 1. Check if PinMe is Installed

```bash
pinme --version
```

If not installed:
```bash
npm install -g pinme
```

### 2. Identify Static Files Directory

Look for these directories (in priority order):
1. `dist/` - Vue/React/Vite default output
2. `build/` - Create React App output
3. `out/` - Next.js static export
4. `public/` - Pure static projects

**Validation rules:**
- Directory must exist
- Must contain `index.html` (for websites)
- Should contain static assets (CSS, JS, images)

### 3. Execute Upload

```bash
pinme upload <directory>
```

Examples:
```bash
pinme upload dist
pinme upload build
pinme upload ./my-static-site
```

### 4. Return Result

After successful upload, return only the preview URL:
```
https://pinme.eth.limo/#/preview/<hash>
```

Users can visit the preview page to:
- View the deployed website
- Get a fixed domain: `https://<name>.pinit.eth.limo`

## Important Rules

**DO:**
- Verify build output exists before uploading
- Check for `index.html` in the upload directory
- Return the preview URL to the user

**DO NOT:**
- Upload `node_modules/`
- Upload `.env` files
- Upload `.git/` directory
- Upload source code (use build output only)
- Upload configuration files (package.json, tsconfig.json, etc.)
- Upload empty or non-existent directories

## Common Workflows

### Build and Deploy (Vue/Vite)
```bash
npm run build
pinme upload dist
```

### Build and Deploy (React CRA)
```bash
npm run build
pinme upload build
```

### Build and Deploy (Next.js Static)
```bash
npm run build
npm run export  # or next export
pinme upload out
```

## Error Handling

| Error | Solution |
|-------|----------|
| `command not found: pinme` | Run `npm install -g pinme` |
| `No such file or directory` | Check path, run build first |
| `Permission denied` | Check folder permissions |
| Upload failed | Check network, retry |

## Other Commands

```bash
# List upload history
pinme list
pinme ls -l 5

# Remove uploaded file
pinme rm <hash>
```

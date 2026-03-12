---
name: push-to-registry
description: Push Packer build metadata to HCP Packer registry for tracking and managing image lifecycle. Use when integrating Packer builds with HCP Packer for version control and governance.
---

# Push to HCP Packer Registry

Configure Packer templates to push build metadata to HCP Packer registry.

**Reference:** [HCP Packer Registry](https://developer.hashicorp.com/hcp/docs/packer)

> **Note:** HCP Packer is free for basic use. Builds push metadata only (not actual images), adding minimal overhead (<1 minute).

## Basic Registry Configuration

```hcl
packer {
  required_version = ">= 1.7.7"
}

variable "image_name" {
  type    = string
  default = "web-server"
}

locals {
  timestamp = regex_replace(timestamp(), "[- TZ:]", "")
}

source "amazon-ebs" "ubuntu" {
  region        = "us-west-2"
  instance_type = "t3.micro"

  source_ami_filter {
    filters = {
      name = "ubuntu/images/*ubuntu-jammy-22.04-amd64-server-*"
    }
    most_recent = true
    owners      = ["099720109477"]
  }

  ssh_username = "ubuntu"
  ami_name     = "${var.image_name}-${local.timestamp}"
}

build {
  sources = ["source.amazon-ebs.ubuntu"]

  hcp_packer_registry {
    bucket_name = var.image_name
    description = "Ubuntu 22.04 base image for web servers"

    bucket_labels = {
      "os"   = "ubuntu"
      "team" = "platform"
    }

    build_labels = {
      "build-time" = local.timestamp
    }
  }

  provisioner "shell" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get upgrade -y",
    ]
  }
}
```

## Authentication

Set environment variables before building:

```bash
export HCP_CLIENT_ID="your-service-principal-client-id"
export HCP_CLIENT_SECRET="your-service-principal-secret"
export HCP_ORGANIZATION_ID="your-org-id"
export HCP_PROJECT_ID="your-project-id"

packer build .
```

### Create HCP Service Principal

1. Navigate to HCP → Access Control (IAM)
2. Create Service Principal
3. Grant "Contributor" role on project
4. Generate client secret
5. Save client ID and secret

## Registry Configuration Options

### bucket_name (required)
The image identifier. Must stay consistent across builds!

```hcl
bucket_name = "web-server"  # Keep this constant
```

### bucket_labels (optional)
Metadata at bucket level. Updates with each build.

```hcl
bucket_labels = {
  "os"        = "ubuntu"
  "team"      = "platform"
  "component" = "web"
}
```

### build_labels (optional)
Metadata for each iteration. Immutable after build completes.

```hcl
build_labels = {
  "build-time" = local.timestamp
  "git-commit" = var.git_commit
}
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Build and Push to HCP Packer

on:
  push:
    branches: [main]

env:
  HCP_CLIENT_ID: ${{ secrets.HCP_CLIENT_ID }}
  HCP_CLIENT_SECRET: ${{ secrets.HCP_CLIENT_SECRET }}
  HCP_ORGANIZATION_ID: ${{ secrets.HCP_ORGANIZATION_ID }}
  HCP_PROJECT_ID: ${{ secrets.HCP_PROJECT_ID }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-packer@main

      - name: Build and push
        run: |
          packer init .
          packer build \
            -var "git_commit=${{ github.sha }}" \
            .
```

## Querying in Terraform

```hcl
data "hcp_packer_artifact" "ubuntu" {
  bucket_name  = "web-server"
  channel_name = "production"
  platform     = "aws"
  region       = "us-west-2"
}

resource "aws_instance" "web" {
  ami           = data.hcp_packer_artifact.ubuntu.external_identifier
  instance_type = "t3.micro"

  tags = {
    PackerBucket = data.hcp_packer_artifact.ubuntu.bucket_name
  }
}
```

## Common Issues

**Authentication Failed**
- Verify HCP_CLIENT_ID and HCP_CLIENT_SECRET
- Ensure service principal has Contributor role
- Check organization and project IDs

**Bucket Name Mismatch**
- Keep `bucket_name` consistent across builds
- Don't include timestamps in bucket_name
- Creates new bucket if name changes

**Build Fails**
- Packer fails immediately if can't push metadata
- Prevents drift between artifacts and registry
- Check network connectivity to HCP API

## Best Practices

- **Consistent bucket names** - Never change for same image type
- **Meaningful labels** - Use for versions, teams, compliance
- **CI/CD automation** - Automate builds and registry pushes
- **Immutable build labels** - Put changing data (git SHA, date) in build_labels

## References

- [HCP Packer Documentation](https://developer.hashicorp.com/hcp/docs/packer)
- [hcp_packer_registry Block](https://developer.hashicorp.com/packer/docs/templates/hcl_templates/blocks/build/hcp_packer_registry)
- [HCP Terraform Provider](https://registry.terraform.io/providers/hashicorp/hcp/latest/docs/data-sources/packer_artifact)

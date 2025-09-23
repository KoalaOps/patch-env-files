# Patch Env Files

Patches environment files (.env) with configuration values from JSON, supporting both single and multiple file updates.

## Features

- 🔧 **Flexible patching** - Update one or multiple .env files in a single action
- 📝 **Base64 support** - Handles encoded JSON patches
- 🎯 **Precise updates** - Add new or override existing environment variables
- 📁 **Auto-create** - Creates files and directories as needed
- 🔄 **Idempotent** - Safe to run multiple times

## Usage

### Single file patching
```yaml
- name: Patch .env file
  uses: KoalaOps/patch-env-files@v1
  with:
    patches: |
      {
        ".env": {
          "API_KEY": "${{ secrets.API_KEY }}",
          "DEBUG": "true"
        }
      }
```

### Multiple files patching
```yaml
- name: Patch multiple env files
  uses: KoalaOps/patch-env-files@v1
  with:
    patches: |
      {
        ".env": {
          "APP_ENV": "production"
        },
        "backend/.env": {
          "DATABASE_URL": "${{ secrets.DB_URL }}",
          "PORT": "3000"
        },
        "frontend/.env": {
          "REACT_APP_API_URL": "https://api.example.com"
        }
      }
```

### With base64 encoded patches
```yaml
- name: Patch with base64 config
  uses: KoalaOps/patch-env-files@v1
  with:
    patches: ${{ secrets.ENV_PATCHES_B64 }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Working directory path | ❌ | '.' |
| `patches` | JSON object mapping file paths to key-value pairs (can be base64 encoded) | ✅ | - |

## Outputs

| Output | Description |
|--------|-------------|
| `updated_files` | Comma-separated list of updated files |

## Patches Format

JSON object mapping file paths to their key-value patches:
```json
{
  ".env": {
    "API_URL": "https://api.production.example.com",
    "LOG_LEVEL": "info"
  },
  "backend/.env": {
    "DATABASE_URL": "postgresql://...",
    "REDIS_URL": "redis://..."
  }
}
```

## Examples

### Dynamic version injection
```yaml
- name: Inject build metadata
  uses: KoalaOps/patch-env-files@v1
  with:
    patches: |
      {
        ".env": {
          "VERSION": "${{ github.sha }}",
          "BUILD_TIME": "${{ github.event.head_commit.timestamp }}",
          "DEPLOYED_BY": "${{ github.actor }}"
        }
      }
```

### Environment-specific configuration
```yaml
- name: Apply environment config
  uses: KoalaOps/patch-env-files@v1
  with:
    path: ./deploy
    patches: |
      {
        "production/.env": {
          "LOG_LEVEL": "error",
          "CACHE_TTL": "3600"
        },
        "staging/.env": {
          "LOG_LEVEL": "debug",
          "CACHE_TTL": "60"
        }
      }
```

### Conditional patching
```yaml
- name: Apply runtime overrides if provided
  if: inputs.runtime_patches_b64 != ''
  uses: KoalaOps/patch-env-files@v1
  with:
    patches: ${{ inputs.runtime_patches_b64 }}
```

## How It Works

1. Decodes base64 patches (if encoded)
2. Validates JSON structure
3. For each file in the patches object:
   - Creates the file if it doesn't exist
   - Creates parent directories if needed
   - Updates existing keys or adds new ones
   - Preserves other keys in the file

## Notes

- Supports both base64 encoded and plain JSON patches
- Files and directories are created automatically if they don't exist
- Existing keys are updated in place, preserving file structure
- Safe to run multiple times (idempotent operation)
- No external dependencies required
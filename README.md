# Patch Env Files

Patches environment files (.env) with configuration values from JSON, typically used for runtime configuration overrides.

## Features

- 🔧 **Environment patching** - Update .env files with new values
- 📝 **Base64 support** - Handles encoded JSON configs
- 🎯 **Targeted updates** - Updates specific overlay directories
- 🔄 **Environment variables** - Inject runtime values
- 📦 **Key-value updates** - Add or override environment variables

## Prerequisites

When using `overlay_dir`, this action requires `npx` (Node.js) to be available in the runner environment. This is available by default on GitHub-hosted runners, but may need to be installed in custom Docker containers.

## Usage

```yaml
- name: Patch environment files
  uses: KoalaOps/patch-env-files@v1
  with:
    config: ${{ inputs.runtime_overrides_b64 }}
    env_file: .env
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `working_directory` | Working directory | ❌ | '.' |
| `config` | Configuration values (base64 encoded JSON or plain JSON) | ✅ | - |
| `overlay_dir` | Overlay directory containing .env files (requires npx) | ❌ | - |
| `env_file` | Specific env file to update | ❌ | '.env' |

## Outputs

| Output | Description |
|--------|-------------|
| `config_json` | Decoded configuration as JSON |
| `updated_files` | List of updated files |

## Configuration Format

Simple key-value JSON object:
```json
{
  "API_URL": "https://api.production.example.com",
  "LOG_LEVEL": "info",
  "FEATURE_FLAGS": "new-ui,analytics"
}
```

## Examples

### Basic Environment Patching
```yaml
- name: Patch production environment
  uses: KoalaOps/patch-env-files@v1
  with:
    config: ${{ secrets.PROD_CONFIG_B64 }}
    env_file: .env
```

### Patching with Dynamic Values
```yaml
- name: Generate patch config
  id: config
  run: |
    CONFIG=$(cat <<EOF | base64 -w0
    {
      "VERSION": "${{ github.sha }}",
      "DEPLOY_TIME": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
      "DEPLOYED_BY": "${{ github.actor }}"
    }
    EOF
    )
    echo "config=$CONFIG" >> $GITHUB_OUTPUT

- name: Patch env file
  uses: KoalaOps/patch-env-files@v1
  with:
    config: ${{ steps.config.outputs.config }}
    env_file: .env
```

### Using overlay_dir with extend-env-files
```yaml
- name: Patch overlay env files
  uses: KoalaOps/patch-env-files@v1
  with:
    config: ${{ inputs.app_config_b64 }}
    overlay_dir: deploy/overlays/production
```

### Conditional Application
```yaml
- name: Patch env file if config provided
  if: inputs.runtime_overrides_b64 != ''
  uses: KoalaOps/patch-env-files@v1
  with:
    config: ${{ inputs.runtime_overrides_b64 }}
    env_file: .env
```

## How It Works

1. Decodes base64 configuration (if encoded)
2. Parses JSON key-value pairs  
3. Patches environment files using one of two methods:
   - **overlay_dir**: Uses `npx extend-env-files` to update .env files
   - **env_file**: Directly updates specified .env file with key-value pairs

## Notes

- Supports both base64 encoded and plain JSON configuration
- Validates JSON structure before applying  
- When using `env_file`, creates backup files during updates
- When using `overlay_dir`, relies on `extend-env-files` tool
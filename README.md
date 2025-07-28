# Ansible Windows PowerShell Downloader

An Ansible playbook that downloads PowerShell scripts from GitHub repositories and executes them on remote Windows servers with configurable parameters, automatic cleanup, and comprehensive error handling.

## Features

- **GitHub Integration**: Download PowerShell scripts directly from GitHub repositories
- **Native Windows Methods**: Uses `win_get_url` for reliable file downloads
- **Configurable Parameters**: Customizable script URLs, destinations, and arguments
- **Execution Policy Management**: Automatically sets appropriate PowerShell execution policies
- **Automatic Cleanup**: Optional cleanup of downloaded scripts after execution
- **Error Handling**: Comprehensive error checking and reporting
- **Flexible Configuration**: Support for various script sources and parameters

## Repository Structure

```
ansible-windows-powershell-downloader/
├── playbooks/
│   └── download-and-execute.yml    # Main playbook
├── inventory/
│   └── hosts.yml                   # Windows server inventory
├── group_vars/
│   └── windows.yml                 # Default configuration
├── examples/
│   ├── run-system-info.yml         # System information example
│   └── manage-service.yml          # Service management example
├── ansible.cfg                     # Ansible configuration
└── README.md
```

## Quick Start

### 1. Prerequisites

- Ansible 4.0+ with Windows support
- WinRM configured on target Windows machines
- Network access to GitHub and target servers

### 2. Configuration

**Update inventory** (`inventory/hosts.yml`):
```yaml
windows:
  hosts:
    your-windows-server:
      ansible_host: 192.168.1.100
      ansible_user: Administrator
      ansible_password: YourPassword
```

**Configure WinRM on Windows servers**:
```powershell
# Run on Windows machines
winrm quickconfig
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```

### 3. Basic Usage

**Simple execution**:
```bash
ansible-playbook playbooks/download-and-execute.yml
```

**With custom parameters**:
```bash
ansible-playbook playbooks/download-and-execute.yml \
  -e powershell_script_url=\"https://raw.githubusercontent.com/user/repo/main/script.ps1\" \
  -e script_dest=\"C:\\Custom\\script.ps1\" \
  -e 'ps_args=[\"-Param1 Value1\", \"-Param2 Value2\"]'
```

## Configuration Variables

### Required Variables
- `powershell_script_url`: URL to the PowerShell script on GitHub
- `script_dest`: Local path where script will be saved
- `ps_args`: Array of arguments to pass to the script

### Optional Variables
| Variable | Default | Description |
|----------|---------|-------------|
| `temp_dir` | `C:\\Temp` | Temporary directory for downloads |
| `ps_execution_policy` | `RemoteSigned` | PowerShell execution policy |
| `cleanup` | `true` | Remove script after execution |
| `timeout` | `60` | Download timeout in seconds |

### Example Variable Configuration

```yaml
# In group_vars/windows.yml or playbook vars
powershell_script_url: \"https://raw.githubusercontent.com/keepithuman/ansible-powershell-automation/main/scripts/Manage-WindowsSystem.ps1\"
script_dest: \"C:\\Temp\\management-script.ps1\"
ps_args:
  - \"-Action SystemInfo\"
  - \"-OutputPath C:\\Reports\"
temp_dir: \"C:\\Automation\"
ps_execution_policy: \"Bypass\"
cleanup: false
timeout: 120
```

## Usage Examples

### Example 1: System Information Collection

```yaml
---
- import_playbook: playbooks/download-and-execute.yml
  vars:
    powershell_script_url: \"https://raw.githubusercontent.com/keepithuman/ansible-powershell-automation/main/scripts/Manage-WindowsSystem.ps1\"
    script_dest: \"C:\\Temp\\system-info.ps1\"
    ps_args:
      - \"-Action SystemInfo\"
      - \"-OutputPath C:\\Reports\"
    cleanup: true
```

**Run it**:
```bash
ansible-playbook examples/run-system-info.yml
```

### Example 2: Service Management

```yaml
---
- import_playbook: playbooks/download-and-execute.yml
  vars:
    powershell_script_url: \"https://raw.githubusercontent.com/keepithuman/ansible-powershell-automation/main/scripts/Manage-WindowsSystem.ps1\"
    script_dest: \"C:\\Temp\\service-management.ps1\"
    ps_args:
      - \"-Action ServiceManagement\"
      - \"-ServiceName Spooler\"
      - \"-ServiceAction Restart\"
    cleanup: true
```

**Run it**:
```bash
ansible-playbook examples/manage-service.yml
```

### Example 3: Custom Script with Multiple Parameters

```bash
ansible-playbook playbooks/download-and-execute.yml \
  -e powershell_script_url=\"https://raw.githubusercontent.com/myorg/scripts/main/custom-script.ps1\" \
  -e script_dest=\"C:\\Scripts\\custom.ps1\" \
  -e 'ps_args=[\"-ComputerName localhost\", \"-Verbose\", \"-OutputFormat JSON\"]' \
  -e ps_execution_policy=\"Bypass\" \
  -e cleanup=false
```

## Playbook Workflow

The main playbook (`download-and-execute.yml`) follows this workflow:

1. **Validation**: Display configuration and validate parameters
2. **Preparation**: Ensure temporary directory exists
3. **Download**: Download PowerShell script from GitHub using `win_get_url`
4. **Verification**: Verify successful download and file integrity
5. **Execution**: Set execution policy and run PowerShell script
6. **Results**: Display execution results and status
7. **Cleanup**: Remove downloaded script (if enabled)
8. **Error Handling**: Fail playbook if script execution fails

## Advanced Configuration

### Multiple Hosts with Different Parameters

```yaml
---
- name: Execute different scripts on different servers
  hosts: windows
  vars:
    scripts_config:
      web-servers:
        url: \"https://raw.githubusercontent.com/org/scripts/main/web-config.ps1\"
        args: [\"-Environment Production\"]
      db-servers:
        url: \"https://raw.githubusercontent.com/org/scripts/main/db-maintenance.ps1\"
        args: [\"-MaintenanceMode true\"]
  
  tasks:
    - include: playbooks/download-and-execute.yml
      vars:
        powershell_script_url: \"{{ scripts_config[group_names[0]].url }}\"
        ps_args: \"{{ scripts_config[group_names[0]].args }}\"
```

### Using Ansible Vault for Sensitive Parameters

```bash
# Create encrypted file
ansible-vault create group_vars/windows/vault.yml

# Add sensitive variables
vault_script_api_key: \"your-api-key\"
vault_database_password: \"secret-password\"

# Use in playbook
ps_args:
  - \"-ApiKey {{ vault_script_api_key }}\"
  - \"-DatabasePassword {{ vault_database_password }}\"
```

## Error Handling

The playbook includes comprehensive error handling:

- **Download Failures**: Checks if script was downloaded successfully
- **Execution Failures**: Monitors PowerShell return codes
- **File Verification**: Validates file existence and integrity
- **Network Issues**: Handles timeout and connectivity problems

**Example error scenarios**:
```bash
# Invalid URL
TASK [Fail if script download failed] *****
fatal: [windows-server-01]: FAILED! => {\"msg\": \"Failed to download PowerShell script from https://invalid-url.com/script.ps1\"}

# Script execution failure
TASK [Fail playbook if script execution failed] *****
fatal: [windows-server-01]: FAILED! => {\"msg\": \"PowerShell script execution failed with return code 1\"}
```

## Troubleshooting

### Common Issues

1. **WinRM Connection Failed**
   ```bash
   # Test connectivity
   ansible windows -m win_ping
   ```

2. **PowerShell Execution Policy**
   ```powershell
   # On Windows machine
   Get-ExecutionPolicy -List
   Set-ExecutionPolicy RemoteSigned -Scope LocalMachine
   ```

3. **Download Timeouts**
   ```yaml
   # Increase timeout
   timeout: 300  # 5 minutes
   ```

4. **SSL Certificate Issues**
   ```yaml
   # In win_get_url task
   validate_certs: no
   ```

### Debug Mode

Run with verbose output for troubleshooting:
```bash
ansible-playbook playbooks/download-and-execute.yml -vvv
```

## Security Considerations

- **Execution Policy**: Uses `RemoteSigned` by default for security
- **HTTPS Downloads**: Always use HTTPS URLs when possible
- **Script Validation**: Verify script sources and content
- **Cleanup**: Enable cleanup to remove downloaded scripts
- **Access Control**: Use proper Windows user permissions

## Performance Optimization

- **Parallel Execution**: Adjust `forks` in `ansible.cfg`
- **Connection Pooling**: Configure WinRM connection settings
- **Download Caching**: Consider local script caching for frequent executions

## Integration Examples

### With CI/CD Pipelines

```yaml
# In GitLab CI or similar
deploy_windows_config:
  script:
    - ansible-playbook playbooks/download-and-execute.yml
        -e powershell_script_url=\"$CI_PROJECT_URL/-/raw/main/deploy/windows-config.ps1\"
        -e 'ps_args=[\"-Environment $CI_ENVIRONMENT_NAME\"]'
```

### With Ansible Tower/AWX

Create job templates with survey prompts for:
- Script URL
- Target servers
- Script parameters
- Execution options

## Contributing

1. Fork the repository
2. Create a feature branch
3. Test your changes on Windows servers
4. Submit a pull request

## License

This project is licensed under the MIT License.

## Related Projects

- [PowerShell Scripts Repository](https://github.com/keepithuman/ansible-powershell-automation)
- [IAG5 Service Integration](https://github.com/keepithuman/iag5-ansible-powershell-service)
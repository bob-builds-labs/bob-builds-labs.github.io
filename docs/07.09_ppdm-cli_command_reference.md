# PPDM CLI Complete Command Reference

## Table of Contents

- [Overview](#overview)
- [Core Commands](#core-commands)
- [Management Commands](#management-commands)
- [Monitoring Commands](#monitoring-commands)
- [Configuration Commands](#configuration-commands)
- [Advanced Commands](#advanced-commands)
- [Specialized Commands](#specialized-commands)
- [Test Results](#test-results)

---

## Overview

The PowerProtect Data Manager (PPDM) CLI provides comprehensive management capabilities for PPDM 20.1. This reference covers all available commands with examples and usage patterns.

### Global Flags

```bash
--debug                    Enable debug/trace mode for API calls and responses
--dry-run                  Show API calls and JSON bodies without executing (secrets redacted)
-h, --help                 Help for any command
--ppdm-server string       Select specific PPDM server from configuration
-v, --version             Show version information
--log-level string        Log level (error, warn, info, debug)
--quiet                   Suppress log messages for clean output
--show-api-calls          Show redacted API requests and responses
--ppdm-log string         Enable OpenTelemetry logging to file or stdout
--otel-enabled            Enable OpenTelemetry observability (traces and metrics)
```

### Output Formats

Most commands support multiple output formats:
- `table` (default) - Human-readable table format
- `json` - Structured JSON output
- `yaml` - YAML format output
- `csv` - CSV format for data export
- `json-raw` - Raw API response JSON (for automation)
- `yaml-raw` - Raw API response YAML (for automation)

---

## Core Commands

### account
Manage PowerProtect Data Manager account operations.

#### Examples
```bash
# List account information
ppdm-cli account list

# Show account details
ppdm-cli account show --output json

# Update account settings
ppdm-cli account update --name "New Account Name"
```

### activities
Monitor PPDM activities and job operations.

#### API Endpoints & Operation IDs
| Command | API Endpoint | Method | Operation ID | API Version |
|---------|-------------|--------|--------------|-------------|
| `activities list` | `/api/v3/activities` | GET | `getAllActivities` | v3 |
| `activities show` | `/api/v3/activities/{id}` | GET | `getActivity` | v3 |
| `activities statistics` | `/api/v3/activities` | GET | `getAllActivities` | v3 |
| `activities monitor` | `/api/v3/activities` | GET | `getAllActivities` | v3 |

#### Examples
```bash
# List all activities
ppdm-cli activities list

# List activities with filtering
ppdm-cli activities list --filter 'status eq "RUNNING"'

# Show activity details
ppdm-cli activities show --id "activity-id"

# Monitor activities in real-time
ppdm-cli activities monitor --filter 'status eq "RUNNING"'

# Export activities to CSV
ppdm-cli activities list --output csv > activities.csv

# Show storage and performance statistics
ppdm-cli activities statistics BACKUP --page-size 50

# Show backup statistics for completed activities
ppdm-cli activities statistics BACKUP --status "COMPLETED"

# Show replication statistics
ppdm-cli activities statistics REPLICATE --output json
```

#### Output Formats
```bash
# Table format (default)
ppdm-cli activities list

# JSON format
ppdm-cli activities list --output json

# JSON-Raw for automation
ppdm-cli activities list --output json-raw
```

#### Statistics Categories
- **BACKUP** - Backup operations with storage statistics (compression, deduplication, transfer rates)
- **REPLICATE** - Replication operations with different metrics

#### Statistics Examples
```bash
# Show backup storage efficiency
ppdm-cli activities statistics BACKUP --search "database"

# Show backup statistics for time period
ppdm-cli activities statistics BACKUP --period "7d"

# Export statistics to JSON for analysis
ppdm-cli activities statistics BACKUP --output json | jq '.[] | {id, assetSizeGB: (.statistics.assetSizeInBytes / 1024 / 1024 / 1024), compressionRatio: (.statistics.reductionPercentage)}'
```

### activities-browser
Interactive browser for PPDM job groups.

#### Examples
```bash
# Launch interactive browser
ppdm-cli activities-browser

# Browser with specific filter
ppdm-cli activities-browser --filter 'status eq "RUNNING"'
```

### activity-cancellations-batch
Cancel multiple activities in batch.

#### Examples
```bash
# Cancel activities by filter
ppdm-cli activity-cancellations-batch cancel --filter 'status eq "RUNNING"'

# Cancel specific activities
ppdm-cli activity-cancellations-batch cancel --activity-ids "id1,id2,id3"

# Dry run to see what would be cancelled
ppdm-cli activity-cancellations-batch cancel --filter 'status eq "RUNNING"' --dry-run
```

### activity-retries-batch
Retry multiple activities in batch.

#### Examples
```bash
# Retry failed activities
ppdm-cli activity-retries-batch retry --filter 'status eq "FAILED"'

# Retry specific activities
ppdm-cli activity-retries-batch retry --activity-ids "id1,id2,id3"

# Dry run to see what would be retried
ppdm-cli activity-retries-batch retry --filter 'status eq "FAILED"' --dry-run
```

### agent-management
Manage PowerProtect Data Manager agent operations.

#### Examples
```bash
# List all agents
ppdm-cli agent-management list

# Show agent details
ppdm-cli agent-management show --id "agent-id"

# Install agent
ppdm-cli agent-management install --host "hostname" --package "agent-package"

# Uninstall agent
ppdm-cli agent-management uninstall --id "agent-id"

# Update agent
ppdm-cli agent-management update --id "agent-id"
```

### alerts
Manage PPDM alerts and notifications.

#### Examples
```bash
# List all alerts
ppdm-cli alerts list

# List critical alerts
ppdm-cli alerts list --filter 'severity eq "CRITICAL"'

# Show alert details
ppdm-cli alerts show --id "alert-id"

# Acknowledge alert
ppdm-cli alerts acknowledge --id "alert-id"

# Export alerts to CSV
ppdm-cli alerts list --output csv > alerts.csv
```

### assets
Manage PPDM assets [DEPRECATED - Use asset-management instead].

#### API Endpoints & Operation IDs
| Command | API Endpoint | Method | Operation ID | API Version |
|---------|-------------|--------|--------------|-------------|
| `assets list` | `/api/v2/assets` | GET | `getAssets` | v2 |
| `assets show` | `/api/v2/assets/{id}` | GET | `getAsset` | v2 |

#### Examples
```bash
# List assets (deprecated)
ppdm-cli assets list

# Show asset details
ppdm-cli assets show --id "asset-id"

# List assets with filtering
ppdm-cli assets list --filter 'type eq "FILE_SYSTEM"'

# Export assets to CSV
ppdm-cli assets list --output csv > assets.csv
```

### backup
Create backup operations with protection policy enforcement.

#### API Endpoints & Operation IDs
| Command | API Endpoint | Method | Operation ID | API Version |
|---------|-------------|--------|--------------|-------------|
| `backup create` | `/api/v2/backups` | POST | `createBackup` | v2 |

#### Examples
```bash
# Create backup with protection policy
ppdm-cli backup create --asset-id "asset-id" --policy-id "policy-id"

# Create backup with custom settings
ppdm-cli backup create \
  --asset-id "asset-id" \
  --policy-id "policy-id" \
  --name "Custom Backup" \
  --description "Backup description"

# Dry run to see backup configuration
ppdm-cli backup create --asset-id "asset-id" --policy-id "policy-id" --dry-run

# Create backup with specific backup level
ppdm-cli backup create --asset-id "asset-id" --policy-id "policy-id" --backup-level FULL
```

### backup-browser
Interactive TUI browser for selecting assets and starting backups.

#### Examples
```bash
# Launch interactive backup browser
ppdm-cli backup-browser

# Browser with specific asset type
ppdm-cli backup-browser --asset-type "FILE_SYSTEM"
```

### backup-restore
Manage PowerProtect Data Manager backup and restore operations.

#### Examples
```bash
# List backup copies
ppdm-cli backup-restore list-copies --asset-id "asset-id"

# Show copy details
ppdm-cli backup-restore show-copy --copy-id "copy-id"

# Restore from copy
ppdm-cli backup-restore restore \
  --copy-id "copy-id" \
  --target-host "target-host" \
  --target-path "/restore/path"

# List restored copies
ppdm-cli backup-restore list-restored-copies

# Show restored copy details
ppdm-cli backup-restore show-restored-copy --id "restored-id"
```

---

## Management Commands

### asset-management
Manage PowerProtect Data Manager asset operations.

#### Examples
```bash
# List all assets
ppdm-cli asset-management list

# List assets by type
ppdm-cli asset-management list --asset-type "FILE_SYSTEM"

# List assets with filtering
ppdm-cli asset-management list --filter 'name co "database"'

# Show asset details
ppdm-cli asset-management show --id "asset-id"

# Create asset
ppdm-cli asset-management create \
  --name "New Asset" \
  --type "FILE_SYSTEM" \
  --host-id "host-id" \
  --path "/path/to/asset"

# Update asset
ppdm-cli asset-management update --id "asset-id" --name "Updated Name"

# Delete asset
ppdm-cli asset-management delete --id "asset-id"
```

### configurations
Manage PPDM appliance configurations.

#### Examples
```bash
# List configurations
ppdm-cli configurations list

# Show configuration details
ppdm-cli configurations show --id "config-id"

# Update configuration
ppdm-cli configurations update --id "config-id" --setting "value"
```

### credentials
Manage PPDM credentials for external systems.

#### Examples
```bash
# List credentials
ppdm-cli credentials list

# Show credential details
ppdm-cli credentials show --id "credential-id"

# Create credential
ppdm-cli credentials create \
  --name "Database Credential" \
  --type "USER_CREDENTIAL" \
  --username "dbuser" \
  --password "password"

# Update credential
ppdm-cli credentials update --id "credential-id" --password "newpassword"

# Delete credential
ppdm-cli credentials delete --id "credential-id"
```

### hosts
Manage PowerProtect Data Manager hosts.

#### API Endpoints & Operation IDs
| Command | API Endpoint | Method | Operation ID | API Version |
|---------|-------------|--------|--------------|-------------|
| `hosts list` | `/api/v2/hosts` | GET | `getHosts` | v2 |
| `hosts show` | `/api/v2/hosts/{id}` | GET | `getHost` | v2 |
| `hosts create` | `/api/v2/hosts` | POST | `createHost` | v2 |
| `hosts update` | `/api/v2/hosts/{id}` | PUT | `updateHost` | v2 |
| `hosts delete` | `/api/v2/hosts/{id}` | DELETE | `deleteHost` | v2 |

#### Examples
```bash
# List all hosts
ppdm-cli hosts list

# List hosts by type
ppdm-cli hosts list --host-type "APPLICATION_HOST"

# Show host details
ppdm-cli hosts show --id "host-id"

# Create host
ppdm-cli hosts create \
  --name "New Host" \
  --type "APPLICATION_HOST" \
  --addresses "192.168.1.100"

# Update host
ppdm-cli hosts update --id "host-id" --name "Updated Host"

# Delete host
ppdm-cli hosts delete --id "host-id"

# List hosts with filtering
ppdm-cli hosts list --filter 'type eq "APP_HOST"'

# Export hosts to CSV
ppdm-cli hosts list --output csv > hosts.csv
```

### infrastructure-objects
Manage infrastructure objects in PowerProtect Data Manager.

#### Examples
```bash
# List all infrastructure objects
ppdm-cli infrastructure-objects list

# List by category with tab completion
ppdm-cli infrastructure-objects list --category <TAB>
ppdm-cli infrastructure-objects list --category STORAGE_SYSTEM

# List by type with tab completion
ppdm-cli infrastructure-objects list --type <TAB>
ppdm-cli infrastructure-objects list --type VMWARE_ESX_HOST

# List by vendor
ppdm-cli infrastructure-objects list --vendor VMWARE

# Combined filtering
ppdm-cli infrastructure-objects list \
  --category STORAGE_SYSTEM \
  --vendor DATADOMAIN \
  --output json

# Show object details
ppdm-cli infrastructure-objects show --id "object-id"

# Export to CSV
ppdm-cli infrastructure-objects list --output csv > infra_objects.csv
```

#### Supported Categories
- `APPLICATION_HOST` - Application server hosts
- `APPLICATION_SYSTEM` - Application system clusters
- `CLOUD_STORAGE_PROFILE` - Cloud storage configurations
- `HYPERVISOR_CLUSTER` - Virtualization clusters
- `HYPERVISOR_MANAGER` - Virtualization management
- `HYPERVISOR_SERVER` - Virtualization servers
- `INVENTORY_SOURCE` - Discovery and inventory sources
- `NETWORK_HOST` - Network-connected hosts
- `STORAGE_APPLIANCE` - Storage appliance devices
- `STORAGE_SYSTEM` - Storage system devices

### inventory-sources
Manage inventory sources in PowerProtect Data Manager.

#### Examples
```bash
# List inventory sources
ppdm-cli inventory-sources list

# List by type
ppdm-cli inventory-sources list --type "VMWARE_VCENTER"

# Show inventory source details
ppdm-cli inventory-sources show --id "source-id"

# Create inventory source
ppdm-cli inventory-sources create \
  --name "vCenter Server" \
  --type "VMWARE_VCENTER" \
  --addresses "vcenter.example.com" \
  --credential-id "cred-id"

# Update inventory source
ppdm-cli inventory-sources update --id "source-id" --name "Updated Name"

# Delete inventory source
ppdm-cli inventory-sources delete --id "source-id"

# Discover assets from inventory source
ppdm-cli inventory-sources discover --id "source-id"
```

### protection-policies
Manage protection policies with enhanced V3 functionality.

#### API Endpoints & Operation IDs
| Command | API Endpoint | Method | Operation ID | API Version |
|---------|-------------|--------|--------------|-------------|
| `protection-policies list` | `/api/v3/protection-policies` | GET | `getProtectionPolicies` | v3 |
| `protection-policies show` | `/api/v3/protection-policies/{id}` | GET | `getProtectionPolicy` | v3 |
| `protection-policies create` | `/api/v3/protection-policies` | POST | `createProtectionPolicy` | v3 |
| `protection-policies update` | `/api/v3/protection-policies/{id}` | PUT | `updateProtectionPolicy` | v3 |
| `protection-policies delete` | `/api/v3/protection-policies/{id}` | DELETE | `deleteProtectionPolicy` | v3 |

#### Examples
```bash
# List protection policies
ppdm-cli protection-policies list

# List active policies
ppdm-cli protection-policies list --filter 'isActive eq true'

# Show policy details
ppdm-cli protection-policies show --id "policy-id"

# Create protection policy
ppdm-cli protection-policies create \
  --name "Daily Backup Policy" \
  --description "Daily backup with 30-day retention" \
  --asset-type "FILE_SYSTEM" \
  --retention-days 30

# Update policy
ppdm-cli protection-policies update --id "policy-id" --name "Updated Policy"

# Delete policy
ppdm-cli protection-policies delete --id "policy-id"

# Search policies by asset type
ppdm-cli protection-policies list --type "FILE_SYSTEM"

# Show policy objectives
ppdm-cli protection-policies show-objective --id "policy-id"
```

### protection-policy-summaries
Manage PowerProtect Data Manager V3 protection policy summaries.

#### Examples
```bash
# List policy summaries
ppdm-cli protection-policy-summaries list

# Show policy summary details
ppdm-cli protection-policy-summaries show --id "summary-id"

# Export summaries to JSON
ppdm-cli protection-policy-summaries list --output json-raw > summaries.json
```

### storage-systems
Manage storage systems in PowerProtect Data Manager.

#### Examples
```bash
# List storage systems
ppdm-cli storage-systems list

# List by type
ppdm-cli storage-systems list --type "DATA_DOMAIN_APPLIANCE"

# Show storage system details
ppdm-cli storage-systems show --id "storage-id"

# Create storage system
ppdm-cli storage-systems create \
  --name "DataDomain System" \
  --type "DATA_DOMAIN_APPLIANCE" \
  --addresses "dd.example.com" \
  --credential-id "cred-id"

# Update storage system
ppdm-cli storage-systems update --id "storage-id" --name "Updated Name"

# Delete storage system
ppdm-cli storage-systems delete --id "storage-id"
```

---

## Monitoring Commands

### health
Monitor PPDM system health.

#### Examples
```bash
# Check system health
ppdm-cli health check

# Get detailed health status
ppdm-cli health status

# Monitor health in real-time
ppdm-cli health monitor

# Export health data
ppdm-cli health check --output json > health.json
```

### health-event-catalogs
Manage health event catalogs.

#### Examples
```bash
# List health event catalogs
ppdm-cli health-event-catalogs list

# Show catalog details
ppdm-cli health-event-catalogs show --id "catalog-id"
```

### metrics
Retrieve PPDM metrics.

#### Examples
```bash
# Get system metrics
ppdm-cli metrics get

# Get specific metric
ppdm-cli metrics get --metric "cpu.usage"

# Export metrics
ppdm-cli metrics get --output json > metrics.json
```

### monitoring-metrics
Monitor PPDM system metrics and health status.

#### Examples
```bash
# Monitor metrics in real-time
ppdm-cli monitoring-metrics monitor

# Get current metrics
ppdm-cli monitoring-metrics current

# Get historical metrics
ppdm-cli monitoring-metrics history --start-time "2024-01-01T00:00:00Z"
```

### monitor
Real-time monitoring of PPDM activities.

#### Examples
```bash
# Monitor all activities
ppdm-cli monitor activities

# Monitor specific activity types
ppdm-cli monitor activities --type "BACKUP"

# Monitor with filtering
ppdm-cli monitor activities --filter 'status eq "RUNNING"'
```

### status
Check PPDM system status.

#### Examples
```bash
# Check system status
ppdm-cli status check

# Get detailed status
ppdm-cli status details

# Check specific component status
ppdm-cli status check --component "database"
```

---

## Configuration Commands

### config
Manage PPDM configuration.

#### Examples
```bash
# List configuration
ppdm-cli config list

# Show current configuration
ppdm-cli config show

# Add new host configuration
ppdm-cli config add-host \
  --name "production" \
  --url "https://ppdm.example.com" \
  --username "admin" \
  --password "password"

# Remove host configuration
ppdm-cli config remove-host --name "production"

# Test connection
ppdm-cli config test --host "production"
```

### completion
Generate completion script for specified shell.

#### Examples
```bash
# Generate bash completion
ppdm-cli completion bash > /etc/bash_completion.d/ppdm-cli

# Generate zsh completion
ppdm-cli completion zsh > ~/.zsh/completion/_ppdm-cli

# Generate fish completion
ppdm-cli completion fish > ~/.config/fish/completions/ppdm-cli.fish

# Generate powershell completion
ppdm-cli completion powershell > ppdm-cli.ps1
```

### common-settings
Manage PPDM common settings.

#### Examples
```bash
# List common settings
ppdm-cli common-settings list

# Show setting details
ppdm-cli common-settings show --key "setting-key"

# Update setting
ppdm-cli common-settings update --key "setting-key" --value "new-value"
```

### identity-providers
Manage identity providers and authentication systems.

#### Examples
```bash
# List identity providers
ppdm-cli identity-providers list

# Show provider details
ppdm-cli identity-providers show --id "provider-id"

# Create identity provider
ppdm-cli identity-providers create \
  --name "LDAP Provider" \
  --type "LDAP" \
  --url "ldap://ldap.example.com"

# Update identity provider
ppdm-cli identity-providers update --id "provider-id" --name "Updated Name"

# Delete identity provider
ppdm-cli identity-providers delete --id "provider-id"
```

### snmp-hosts
Manage SNMP hosts.

#### Examples
```bash
# List SNMP hosts
ppdm-cli snmp-hosts list

# Show SNMP host details
ppdm-cli snmp-hosts show --id "snmp-host-id"

# Create SNMP host
ppdm-cli snmp-hosts create \
  --name "SNMP Monitor" \
  --address "192.168.1.100" \
  --community "public"

# Update SNMP host
ppdm-cli snmp-hosts update --id "snmp-host-id" --community "newcommunity"

# Delete SNMP host
ppdm-cli snmp-hosts delete --id "snmp-host-id"
```

### snmp-service
Manage SNMP service configuration.

#### Examples
```bash
# Show SNMP service status
ppdm-cli snmp-service status

# Configure SNMP service
ppdm-cli snmp-service configure \
  --enabled true \
  --port 162 \
  --community "public"

# Test SNMP configuration
ppdm-cli snmp-service test
```

---

## Advanced Commands

### curl
Direct API access to any PPDM endpoint.

#### Examples
```bash
# GET request with pagination
ppdm-cli curl --endpoint activities --method GET --apiver api/v2

# POST request with JSON body
ppdm-cli curl --endpoint protection-policies --method POST --apiver api/v2 \
  --body '{"name":"Test Policy","assetType":"FILE_SYSTEM"}'

# PUT request
ppdm-cli curl --endpoint hosts/host-id --method PUT --apiver api/v2 \
  --body '{"name":"Updated Host"}'

# DELETE request
ppdm-cli curl --endpoint activities/activity-id --method DELETE --apiver api/v2

# Request with filtering
ppdm-cli curl --endpoint activities --method GET --apiver api/v2 \
  --filter 'status eq "RUNNING"'

# Request with pagination
ppdm-cli curl --endpoint activities --method GET --apiver api/v2 \
  --page 1 --size 50

# Request with specific output format
ppdm-cli curl --endpoint activities --method GET --apiver api/v2 \
  --output json-raw
```

### datadomain-mtrees
Manage PowerProtect Data Domain MTrees.

#### Examples
```bash
# List MTrees
ppdm-cli datadomain-mtrees list

# List MTrees for specific storage system
ppdm-cli datadomain-mtrees list --storage-system-id "storage-id"

# Show MTree details
ppdm-cli datadomain-mtrees show --id "mtree-id"

# Create MTree
ppdm-cli datadomain-mtrees create \
  --name "Backup MTree" \
  --storage-system-id "storage-id" \
  --path "/backup/mtree"

# Update MTree
ppdm-cli datadomain-mtrees update --id "mtree-id" --name "Updated MTree"

# Delete MTree
ppdm-cli datadomain-mtrees delete --id "mtree-id"
```

### kubernetes
Manage Kubernetes clusters.

#### Examples
```bash
# List Kubernetes clusters
ppdm-cli kubernetes list

# Show cluster details
ppdm-cli kubernetes show --id "cluster-id"

# Create cluster
ppdm-cli kubernetes create \
  --name "Production Cluster" \
  --api-server "https://k8s.example.com" \
  --credential-id "k8s-cred"

# Update cluster
ppdm-cli kubernetes update --id "cluster-id" --name "Updated Cluster"

# Delete cluster
ppdm-cli kubernetes delete --id "cluster-id"
```

### migrate
Migrate exported copy to another datastore.

#### Examples
```bash
# List available copies for migration
ppdm-cli migrate list-copies

# Migrate copy to new datastore
ppdm-cli migrate execute \
  --copy-id "copy-id" \
  --target-datastore-id "datastore-id" \
  --target-path "/migrated"

# Dry run migration
ppdm-cli migrate execute \
  --copy-id "copy-id" \
  --target-datastore-id "datastore-id" \
  --dry-run
```

### verify-cloud-storage
Verify cloud storage profile connection.

#### Examples
```bash
# Verify cloud storage profile
ppdm-cli verify-cloud-storage --profile-id "profile-id"

# Verify with detailed output
ppdm-cli verify-cloud-storage --profile-id "profile-id" --output json

# Test connection only
ppdm-cli verify-cloud-storage --profile-id "profile-id" --test-only
```

### active-directory
Manage Active Directory integration and authentication.

#### Examples
```bash
# List Active Directory configuration
ppdm-cli active-directory list

# Show Active Directory details
ppdm-cli active-directory show

# Configure Active Directory
ppdm-cli active-directory configure \
  --url "ldap://ad.example.com" \
  --domain "example.com" \
  --username "admin" \
  --password "password"

# Test Active Directory connection
ppdm-cli active-directory test
```

### activity
Unified activity management with enhanced V3 functionality.

#### API Endpoints & Operation IDs
| Command | API Endpoint | Method | Operation ID | API Version |
|---------|-------------|--------|--------------|-------------|
| `activity list` | `/api/v3/activities` | GET | `getAllActivities` | v3 |
| `activity show` | `/api/v3/activities/{id}` | GET | `getActivity` | v3 |
| `activity statistics` | `/api/v3/activities` | GET | `getAllActivities` | v3 |

#### Examples
```bash
# List activities with enhanced filtering
ppdm-cli activity list --filter 'status eq "RUNNING"'

# Show activity details with full context
ppdm-cli activity show --id "activity-id" --output json

# Get activity statistics
ppdm-cli activity statistics BACKUP --period "7d"

# Monitor activities in real-time
ppdm-cli activity monitor --filter 'status eq "RUNNING"'
```

### ai
AI assistant for PPDM operations and analysis.

#### Examples
```bash
# Get AI assistance for backup analysis
ppdm-cli ai analyze --type backup --asset-id "asset-id"

# Get AI recommendations
ppdm-cli ai recommend --category optimization

# Generate AI reports
ppdm-cli ai report --type performance --period "30d"

# AI-powered troubleshooting
ppdm-cli ai troubleshoot --activity-id "activity-id"
```

### audit-compliance
Manage audit trails and compliance reporting.

#### Examples
```bash
# List audit events
ppdm-cli audit-compliance list --start-time "2024-01-01T00:00:00Z"

# Show compliance status
ppdm-cli audit-compliance status --category "data-protection"

# Generate compliance report
ppdm-cli audit-compliance report --format pdf --output compliance-report.pdf

# Export audit data
ppdm-cli audit-compliance export --format csv > audit-data.csv
```

### certificates
Manage SSL/TLS certificates for PPDM.

#### Examples
```bash
# List certificates
ppdm-cli certificates list

# Show certificate details
ppdm-cli certificates show --id "cert-id"

# Upload certificate
ppdm-cli certificates upload \
  --file "certificate.crt" \
  --key "private.key" \
  --name "Web Certificate"

# Renew certificate
ppdm-cli certificates renew --id "cert-id"

# Delete certificate
ppdm-cli certificates delete --id "cert-id"
```

### discoveries
Manage discovery operations for inventory sources.

#### Examples
```bash
# List discovery operations
ppdm-cli discoveries list

# Create discovery task
ppdm-cli discoveries create --source-id "source-id"

# Show discovery details
ppdm-cli discoveries show --id "discovery-id"

# Monitor discovery progress
ppdm-cli discoveries monitor --id "discovery-id"

# Cancel discovery
ppdm-cli discoveries cancel --id "discovery-id"
```

### identity-provider
Manage identity provider configurations.

#### Examples
```bash
# List identity providers
ppdm-cli identity-provider list

# Show provider details
ppdm-cli identity-provider show --id "provider-id"

# Create LDAP provider
ppdm-cli identity-provider create \
  --name "LDAP Provider" \
  --type "LDAP" \
  --url "ldap://ldap.example.com"

# Update provider
ppdm-cli identity-provider update --id "provider-id" --name "Updated Name"

# Delete provider
ppdm-cli identity-provider delete --id "provider-id"
```

### infrastructure-nodes
Manage infrastructure nodes in PPDM.

#### Examples
```bash
# List infrastructure nodes
ppdm-cli infrastructure-nodes list

# Show node details
ppdm-cli infrastructure-nodes show --id "node-id"

# List nodes by type
ppdm-cli infrastructure-nodes list --node-type "STORAGE_NODE"

# Monitor node health
ppdm-cli infrastructure-nodes health --id "node-id"
```

### nodes
Manage PPDM cluster nodes.

#### Examples
```bash
# List cluster nodes
ppdm-cli nodes list

# Show node status
ppdm-cli nodes show --id "node-id"

# Monitor node performance
ppdm-cli nodes monitor --id "node-id"

# Get node metrics
ppdm-cli nodes metrics --id "node-id" --period "1h"
```

### protection-configurations
Manage protection configurations and settings.

#### Examples
```bash
# List protection configurations
ppdm-cli protection-configurations list

# Show configuration details
ppdm-cli protection-configurations show --id "config-id"

# Create protection configuration
ppdm-cli protection-configurations create \
  --name "Production Config" \
  --asset-type "FILE_SYSTEM" \
  --retention-days 30

# Update configuration
ppdm-cli protection-configurations update --id "config-id" --name "Updated Config"
```

### storage
Manage storage operations and configurations.

#### Examples
```bash
# List storage resources
ppdm-cli storage list

# Show storage details
ppdm-cli storage show --id "storage-id"

# Get storage metrics
ppdm-cli storage metrics --id "storage-id"

# Analyze storage usage
ppdm-cli storage analyze --period "7d"
```

### syslogs
Manage system logs and logging configuration.

#### Examples
```bash
# List system logs
ppdm-cli syslogs list

# Show log details
ppdm-cli syslogs show --id "log-id"

# Export logs
ppdm-cli syslogs export --start-time "2024-01-01T00:00:00Z" > logs.json

# Configure logging
ppdm-cli syslogs configure --level "INFO" --destination "file"
```

### upgrade
Manage PPDM appliance upgrades.

#### Examples
```bash
# Check upgrade status
ppdm-cli upgrade status

# List available upgrades
ppdm-cli upgrade list

# Start upgrade process
ppdm-cli upgrade start --version "20.1.1.0"

# Monitor upgrade progress
ppdm-cli upgrade monitor

# Rollback upgrade
ppdm-cli upgrade rollback --version "20.1.0.0"
```

### verify-cloud-storage
Verify cloud storage profile connection.

#### Examples
```bash
# Verify cloud storage profile
ppdm-cli verify-cloud-storage --profile-id "profile-id"

# Verify with detailed output
ppdm-cli verify-cloud-storage --profile-id "profile-id" --output json

# Test connection only
ppdm-cli verify-cloud-storage --profile-id "profile-id" --test-only
```

### verify-object-storage
Verify object storage profile connection.

#### Examples
```bash
# Verify object storage profile
ppdm-cli verify-object-storage --profile-id "profile-id"

# Verify with detailed output
ppdm-cli verify-object-storage --profile-id "profile-id" --output json

# Test connection only
ppdm-cli verify-object-storage --profile-id "profile-id" --test-only
```

---

## Specialized Commands

### avamar
Manage Avamar inventory sources and assets.

#### Examples
```bash
# List Avamar inventory sources
ppdm-cli avamar list-sources

# List Avamar assets
ppdm-cli avamar list-assets --source-id "source-id"

# Show Avamar asset details
ppdm-cli avamar show-asset --asset-id "asset-id"
```

### copies
Analyze PPDM backup copies.

#### Examples
```bash
# List backup copies
ppdm-cli copies list

# List copies by asset
ppdm-cli copies list --asset-id "asset-id"

# Show copy details
ppdm-cli copies show --copy-id "copy-id"

# Analyze copy usage
ppdm-cli copies analyze --copy-id "copy-id"
```

### copies-browser
Interactive TUI browser for browsing and sorting backup copies.

#### Examples
```bash
# Launch interactive copies browser
ppdm-cli copies-browser

# Browser with specific filter
ppdm-cli copies-browser --filter 'assetType eq "FILE_SYSTEM"'
```

### resource-group
Manage PPDM resource groups.

#### Examples
```bash
# List resource groups
ppdm-cli resource-group list

# Show resource group details
ppdm-cli resource-group show --id "group-id"

# Create resource group
ppdm-cli resource-group create \
  --name "Production Resources" \
  --description "Production environment resources"

# Update resource group
ppdm-cli resource-group update --id "group-id" --name "Updated Group"

# Delete resource group
ppdm-cli resource-group delete --id "group-id"
```

---

## Test Results

### Comprehensive Test Results

All commands have been tested with various scenarios and output formats. Test results are documented in separate files:

- [FUNCTIONALITY_TEST_RESULTS.md](../FUNCTIONALITY_TEST_RESULTS.md) - Core functionality tests
- [INFRASTRUCTURE_OBJECTS_IMPLEMENTATION_REPORT.md](../INFRASTRUCTURE_OBJECTS_IMPLEMENTATION_REPORT.md) - Infrastructure objects tests
- [JSON_RAW_COMPLETE_IMPLEMENTATION_REPORT.md](../JSON_RAW_COMPLETE_IMPLEMENTATION_REPORT.md) - JSON output format tests
- [YAML_RAW_COMPLETE_IMPLEMENTATION_REPORT.md](../YAML_RAW_COMPLETE_IMPLEMENTATION_REPORT.md) - YAML output format tests

### Test Coverage

#### ✅ Core Functionality Tests
- Command execution and error handling
- Output format validation (table, json, yaml, csv, json-raw, yaml-raw)
- Filter and pagination functionality
- Authentication and connection testing

#### ✅ Integration Tests
- Multi-host configuration support
- Container environment compatibility
- Environment variable configuration
- Tab completion functionality

#### ✅ Performance Tests
- Large dataset handling
- Pagination efficiency
- Memory usage optimization
- Response time validation

#### ✅ Edge Case Tests
- Invalid parameter handling
- Network error recovery
- Authentication failure scenarios
- Malformed response handling

### Test Execution Examples

```bash
# Run comprehensive functionality tests
./test_functionality.sh

# Run Oracle database specific tests
./test_oracle_database.sh

# Run debug mode tests
./test_debug.sh

# Quick validation test
./quick-test.sh
```

---

## Usage Patterns

### Common Workflows

#### 1. Daily Operations
```bash
# Check system status
ppdm-cli status check

# Monitor running activities
ppdm-cli activities list --filter 'status eq "RUNNING"'

# Review recent alerts
ppdm-cli alerts list --filter 'severity eq "CRITICAL"'
```

#### 2. Backup Management
```bash
# Create backup
ppdm-cli backup create --asset-id "asset-id" --policy-id "policy-id"

# Monitor backup progress
ppdm-cli monitor activities --type "BACKUP"

# Review backup copies
ppdm-cli backup-restore list-copies --asset-id "asset-id"
```

#### 3. Infrastructure Management
```bash
# List storage systems
ppdm-cli infrastructure-objects list --category STORAGE_SYSTEM

# Monitor storage health
ppdm-cli health check

# Review storage metrics
ppdm-cli metrics get --metric "storage.usage"
```

#### 4. Automation Scripts
```bash
#!/bin/bash
# Export all data for reporting
ppdm-cli activities list --output csv > activities.csv
ppdm-cli alerts list --output csv > alerts.csv
ppdm-cli infrastructure-objects list --output csv > infra.csv

# Generate health report
ppdm-cli health check --output json > health.json
```

### Best Practices

1. **Use json-raw for automation**: Provides clean, unparsed API responses
2. **Filter at the source**: Use API filters instead of client-side filtering
3. **Pagination for large datasets**: Use page/size parameters for efficiency
4. **Dry-run for testing**: Use --dry-run flag to preview operations
5. **Tab completion**: Use <TAB> to discover available options

---

## Troubleshooting

### Common Issues

#### Authentication Problems
```bash
# Test connection
ppdm-cli config test --host "hostname"

# Check configuration
ppdm-cli config list

# Use debug mode for detailed errors
ppdm-cli activities list --debug
```

#### Output Format Issues
```bash
# Use json-raw for clean automation output
ppdm-cli activities list --output json-raw 2>/dev/null

# Redirect stderr to avoid pollution
ppdm-cli activities list --output json-raw 2>/dev/null | jq '.'
```

#### Performance Issues
```bash
# Use pagination for large datasets
ppdm-cli activities list --page 1 --size 50

# Filter at API level
ppdm-cli activities list --filter 'status eq "RUNNING"'
```

### Debug Mode

Enable debug mode for detailed API information:
```bash
# Enable debug for all commands
ppdm-cli --debug activities list

# Show API calls without execution
ppdm-cli --dry-run backup create --asset-id "asset-id" --policy-id "policy-id"
```

---

## Version Information

```bash
# Check CLI version
ppdm-cli version

# Check PPDM version
ppdm-cli status check --output json | jq '.version'
```

---

*This documentation covers all available commands in the PPDM CLI. For command-specific help, use `ppdm-cli <command> --help`.*

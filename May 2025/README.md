# Tip of the Month - May 2025

## Check List of Deployments Having DEBUG Logging Enabled

This tip helps you identify deployments that have DEBUG logging enabled, which is crucial for preventing log overflow in Elasticsearch.

### The Problem
Deployments with DEBUG logging enabled can create excessive logs that clog up Elasticsearch, leading to performance issues and storage problems.

### The Solution
Use this script to check for deployments with DEBUG logging:

```bash
result=$(curl -s 'https://houston.basedomain.io/v1' \
  -H 'Accept-Encoding: gzip, deflate, br' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Connection: keep-alive' \
  -H 'DNT: 1' \
  -H 'Origin: https://houston.basedomain.io' \
  -H 'authorization: <api-token>' \
  --data-binary '{"query":"query GetDeploymentsWithEnvVars {\n  deployments {\n    id\n    label\n    releaseName\n    environmentVariables {\n      key\n      value\n      isSecret\n    }\n  }\n}"}' \
  --compressed | jq -r '.data.deployments[]? | select(.environmentVariables[]? | .key == "AIRFLOW__LOGGING__LOGGING_LEVEL" and .value == "DEBUG") | .releaseName')

if [ -z "$result" ]; then
  echo "nothing found"
else
  echo "$result"
fi
```

### How It Works
1. The script makes a GraphQL query to the Houston API to fetch all deployments and their environment variables
2. It filters for deployments where `AIRFLOW__LOGGING__LOGGING_LEVEL` is set to `DEBUG`
3. Returns the release names of deployments with DEBUG logging enabled
4. If no deployments are found, it reports "nothing found"

### Important Notes
- Replace `<api-token>` with your actual API token
- Replace `basedomain.io` with your actual domain
- Requires `jq` to be installed for JSON parsing
- This check should be performed regularly to maintain system health

### Why This Matters
DEBUG logging can significantly impact:
- **Storage costs**: Excessive logs consume more storage
- **Search performance**: Elasticsearch performance degrades with too much data
- **Log retention**: Important logs may be rotated out faster
- **Monitoring efficiency**: Signal-to-noise ratio decreases
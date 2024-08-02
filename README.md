- 👋 Hi, I’m @LeojMarco
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
LeojMarco/LeojMarco is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
#!/bin/bash
# ... (Copyright and License headers remain the same)

if [[ $UNDER_TEST_RUNNER -ne 1 ]]; then
  echo "This script should be run under run_integration_tests.sh"
  exit 1
fi

set -euo pipefail

log() { echo "[$(date)] $1"; }

# Run bazel to populate some of the metrics.
log "Running Bazel test..."
bazel --output_base="$BAZEL_CACHE_DIR" test --config self_test //:dummy_test

# Fetch metrics (with error handling and logging)
log "Fetching metrics..."
all_contents="$(curl --retry 5 --retry-delay 0 --retry-max-time 30 http://127.0.0.1:50061/metrics 2>&1)" || {
  log "Failed to fetch metrics: $all_contents"
  exit 1
}

echo "$all_contents"

# ... (Rest of the metric validations - with potential for more explicit checks and logging)

I'm- 👋 Hi, I’m @LeojMarco
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
#!/bin/bash

# ... (Copyright and License headers remain the same)

if [[ $UNDER_TEST_RUNNER -ne 1 ]]; then
  echo "This script should be run under run_integration_tests.sh"
  exit 1
fi

set -euo pipefail

log() { echo "[$(date)] $1"; }

run_bazel_test() {
  log "Running Bazel test..."
  bazel --output_base="$BAZEL_CACHE_DIR" test --config self_test //:dummy_test
}

fetch_metrics() {
  log "Fetching metrics..."
  local metrics_url="http://127.0.0.1:50061/metrics"
  all_contents="$(curl --retry 5 --retry-delay 0 --retry-max-time 30 "$metrics_url" 2>&1)" || {
    log "Failed to fetch metrics from $metrics_url: $all_contents"
    exit 1
  }
  echo "$all_contents"
}

check_metric() {
  local metric_name=$1
  local expected_value=$2

  log "Checking metric: $metric_name = $expected_value"
  if ! grep -q "$metric_name $expected_value" <<< "$all_contents"; then
    log "ERROR: Expected metric '$metric_name' to be '$expected_value' but not found in metrics."
    exit 1
  fi
}

# Main script execution
run_bazel_test
all_contents=$(fetch_metrics)

echo "$all_contents"

# Static Metric Validations (with explicit checks and logging)
check_metric 'nativelink_stores_AC_MAIN_STORE_evicting_map_max_bytes' 500000000
check_metric 'nativelink_stores_AC_MAIN_STORE_read_buff_size_bytes' 32768

# Uniqueness Check (with logging)
count=$(grep -c 'nativelink_stores_AC_MAIN_STORE_evicting_map_max_bytes 500000000' <<< "$all_contents")
if [[ $count -ne 1 ]]; then
  log "ERROR: Expected 1 instance of CAS_MAIN_STORE, but found $count"
  exit 1
fi

# Dynamic Metric Validations (with explicit checks and logging - adjust as needed)
check_metric 'nativelink_stores_AC_MAIN_STORE_evicting_map_item_size_bytes{quantile="0.99"}' '[0-9]+(\.[0-9]+)?' # Check for a numeric value
check_metric 'nativelink_stores_AC_MAIN_STORE_evicting_map_items_in_store_total' 3

log "All metric checks passed successfully."

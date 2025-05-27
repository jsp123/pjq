# pjq - Parallel jq processor

Drop-in replacement for jq with automatic parallel processing for faster execution on large JSON files.

## Features

- **6x faster** on large datasets (automatic parallel processing)
- **Drop-in replacement** - identical results to regular jq
- **Smart detection** - automatically chooses parallel vs sequential based on query complexity
- **Handles aggregations** - count, sum, average with parallel processing
- **Control options** - force parallel/sequential modes when needed

## About

I built pjq to speed up my own data processing workflows with large JSON files. It gives me consistent 6x performance improvements on my datasets. Your mileage may vary - please test with your data first.

**Not extensively tested on exotic jq features.** Contributions welcome!

## Installation

### Prerequisites
```bash
# Install dependencies
sudo apt install jq parallel  # Ubuntu/Debian
brew install jq parallel      # macOS
```

### Quick Install (Local)
```bash
# Download and make executable
curl -O https://raw.githubusercontent.com/yourusername/pjq/main/pjq
chmod +x pjq

# Use with ./pjq
./pjq 'select(.price > 50)' data.jsonl
```

### System-wide Install
```bash
# Download pjq
curl -O https://raw.githubusercontent.com/yourusername/pjq/main/pjq
chmod +x pjq

# Install to system PATH
sudo cp pjq /usr/local/bin/pjq

# Now available globally
pjq 'select(.price > 50)' data.jsonl
```

### Verify Installation
```bash
# Test basic functionality
pjq --version
pjq --help

# Test with sample data
echo '{"name":"test","price":100}' | pjq 'select(.price > 50)'
```

## Quick Start

```bash
# Basic usage - just replace 'jq' with 'pjq'
pjq 'select(.price > 50)' large_dataset.jsonl

# Automatically goes parallel on large files
pjq 'select(.status == "active")' huge_file.jsonl | wc -l

# Complex queries stay sequential (for correctness)
pjq 'group_by(.category) | map(length)' data.jsonl
```

## Performance Examples

Based on real-world testing with datasets up to 1M+ records:

```bash
# Small files (< 1000 lines): Sequential processing
pjq 'select(.active == true)' small_data.jsonl
# ~0.05s (same as jq)

# Medium files (5k+ lines): Automatic parallel processing  
pjq 'select(.price > 100)' medium_data.jsonl | wc -l
# ~0.12s vs jq's ~0.50s (4x faster)

# Large files (800k+ lines): Maximum parallel benefit
pjq 'select(.amount > 50)' large_data.jsonl | wc -l
# ~0.42s vs jq's ~2.55s (6x faster!)
```

## Supported Query Types

### ✅ Parallel Processing (Fast)

```bash
# Simple filters
pjq 'select(.price > 100)' data.jsonl
pjq 'select(.status != null and .active == true)' data.jsonl

# Basic aggregations  
pjq '[select(.price > 50)] | length' data.jsonl          # Count
pjq '[select(.price > 50) | .price] | add' data.jsonl    # Sum
pjq '[select(.price > 50) | .price] | add/length' data.jsonl  # Average
```

### ⚠️ Sequential Processing (Accurate)

```bash
# Complex operations that need global context
pjq 'group_by(.category)' data.jsonl
pjq 'sort_by(.timestamp)' data.jsonl  
pjq 'unique_by(.id)' data.jsonl
```

## Advanced Options

```bash
# Force parallel processing (advanced users)
pjq --parallel 'select(.price > 50)' data.jsonl

# Force sequential processing  
pjq --sequential 'select(.price > 50)' data.jsonl

# Skip output ordering for maximum I/O performance
pjq --no-order 'select(.price > 50)' data.jsonl

# Debug mode to see processing decisions
pjq --debug 'select(.price > 50)' data.jsonl
```

## Use Cases

### Data Processing Pipelines
```bash
# ETL workflows - 6x faster filtering
pjq 'select(.date >= "2025-01-01")' events.jsonl | process_data.py

# Log analysis - handle large log files efficiently  
pjq 'select(.level == "ERROR")' application.logs | alert_system.sh

# Data validation - quick filtering of large datasets
pjq 'select(.email != null and (.email | test("@")))' users.jsonl
```

### Aggregation & Analytics
```bash
# Count matching records
pjq '[select(.active == true)] | length' users.jsonl

# Calculate totals
pjq '[select(.status == "paid") | .amount] | add' transactions.jsonl

# Compute averages  
pjq '[select(.rating > 0) | .rating] | add/length' reviews.jsonl
```

## Testing & Compatibility

Built and tested on real datasets:
- Files up to 1M+ records
- Null value handling  
- Numeric comparisons
- String operations
- Stdin/pipe processing
- Multiple file processing

## Performance Notes

- **Threshold**: Files > 1000 lines trigger parallel processing
- **CPU Usage**: Utilizes all available cores automatically  
- **Memory**: Efficient - processes data in chunks, no full file loading
- **Ordering**: --keep-order adds ~7% processing time for deterministic output
- **Scalability**: Tested on 800k+ record datasets with consistent 6x speedup
- **I/O Impact**: --keep-order uses disk buffering (use --no-order on I/O-constrained systems)

## Contributing

Found a bug? Want to add test coverage? Pull requests welcome!

```bash
# Run the test suite
./test_pjq.sh

# Test specific scenarios
pjq --debug 'your_query_here' your_data.jsonl
```

## License

MIT License - see LICENSE file for details.

---

**pjq** - Because life's too short for slow JSON processing! ⚡
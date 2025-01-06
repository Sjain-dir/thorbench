# Solana RPC Benchmark Tool

A benchmarking tool for Solana RPC nodes using Raydium swaps. While simple SOL transfers only use about 200-5000 compute units (CU), this tool performs Raydium swaps that use ~30,000 CU to better simulate real-world RPC node performance under load.

## Why This Tool?

### Better Performance Testing
- Simple SOL transfers are too lightweight for meaningful RPC testing
- Raydium swaps use ~30,000 CU (vs 200-5000 CU for transfers)
- Better represents real DeFi transaction workloads
- Tests node performance with complex instructions

### Technical Details
- Solana block limit: ~1.4M CU per block
- Raydium swap: ~30,000 CU per transaction
- Max ~40-45 swap transactions per block
- Tests both compute-heavy and data-heavy operations
- Measures end-to-end transaction performance:
  - Send latency
  - Block inclusion time
  - Transaction success rates
  - Landing time distribution
  - Block distribution analysis

### Real-World Simulation
- Tests node performance under DeFi-like load
- Verifies WebSocket notification reliability
- Measures transaction finality times
- Identifies potential bottlenecks

## ⚠️ Important Warnings

This tool executes real swaps on Raydium with real funds:
- **ONLY use a dedicated test wallet with limited funds**
- **Real tokens will be swapped**
- **Real fees will be paid**
- **Incorrect settings can cause significant losses**
- **Start with small test amounts**

## Installation

### Option 1: Download Binary (Recommended)

1. Go to the [Releases](https://github.com/thorlabsDev/thorbench/releases) tab
2. Download for your platform:
  - `thorbench-linux-amd64` for Linux
  - `thorbench-windows-amd64.exe` for Windows
  - `thorbench-darwin-amd64` for macOS
3. For Linux/macOS: `chmod +x thorbench-*`

### Option 2: Build from Source

Requires Go 1.20 or higher:
```bash
git clone https://github.com/your-repo/thorbench.git
cd thorbench
go mod tidy
go build
```

## Configuration

The tool creates a `config.json` template on first run:

```json
{
  "private_key": "<YOUR-BASE58-PRIVATE-KEY>",
  "rpc_url": "https://api.mainnet-beta.solana.com",
  "ws_url": "",  // Optional: WebSocket URL
  "send_rpc_url": "",  // Optional: Separate RPC for sending
  "rate_limit": 200,
  "tx_count": 100,
  "prio_fee": 1.0,
  "node_retries": 3,
  "input_mint": "So11111111111111111111111111111111111111112",  // SOL
  "output_mint": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",  // USDC
  "slippage": 100.0,  // Use 100% for testing
  "amount": 0.1,  // Amount per swap
  "compute_unit_limit": 30000,  // Optional: defaults to 30000
  "skip_warmup": false  // Optional: whether to skip warmup transaction
}
```

### Warmup Transaction

The tool performs a warmup transaction by default before starting the benchmark. This helps:
- Verify pool configuration and connectivity
- Ensure proper token account setup
- Test transaction parameters
- Prime any caches in the node

When to disable warmup (`"skip_warmup": true`):
- If token accounts are already pre-created
- During rapid sequential tests
- When testing cold-start performance
- When testing failover scenarios
- If pool parameters are already verified

### Critical Settings

#### Pool Selection
- **ONLY use major pools** with high liquidity (>$1M):
  - SOL/USDC
  - SOL/RAY
  - BONK/SOL etc.
- **NEVER use**:
  - Small pools (<$100k liquidity)
  - New/unverified tokens
  - Low volume pools
  - Highly volatile pairs

#### Performance Settings
- `rate_limit`: Maximum transactions per second
- `node_retries`: Number of RPC retries per transaction
- `compute_unit_limit`: Maximum compute units per transaction
- `prio_fee`: Priority fee in lamports per compute unit

#### Slippage and Amount
- Set slippage to 100.0 (100%) for testing
  - Lower values will cause "slippage limit exceeded" errors
  - Only safe with major pools
  - Can cause large losses in small pools
- Start with small amounts
- Check pool liquidity before setting amount

#### Required Funds
Test wallet needs:
- SOL for fees (base fee + priority fee × compute units)
- Input tokens (amount × number of transactions)
- Extra for potential retries

## Usage

1. Configure `config.json`
2. Verify wallet balances
3. Run: `./thorbench`

The tool outputs:
- Success/failure rates
- Landing time statistics
- Block distribution analysis
- Timing statistics (min/max/avg/median/percentiles)
- Logs in `thor_bench_<timestamp>_<testid>.log`

## Output Analysis

The benchmark provides detailed statistics:

### Core Statistics
- Total transactions attempted
- Successful transactions
- Failed transactions (slippage vs other failures)
- Not landed or timed-out transactions

### Success Rates
- Effective success rate (successful/total configured)
- Overall landing rate (landed/attempted)

### Timing Statistics (per transaction status)
- Minimum time
- Maximum time
- Average time
- Median time
- 90th/95th/99th percentiles

### Block Distribution
- Transactions per block
- Block spacing analysis
- Landing time distribution

## Troubleshooting

### Insufficient Balance
- Check SOL for fees
- Ensure enough input tokens
- Account for priority fees

### Slippage Failures
- Verify using large enough pool
- Check token pair liquidity
- Reduce amount per transaction

### Missing Transactions
- Verify RPC stability
- Lower rate limit
- Increase retries
- Check WebSocket connection

## Best Practices

1. Test with minimal amounts first
2. Use dedicated test wallet
3. Monitor market conditions
4. Keep transaction logs
5. Never share private keys
6. Start with warmup enabled
7. Monitor resource usage

## License
Apache License 2.0 - See LICENSE file for details.

## Disclaimer
This software is provided "as is", without warranty of any kind, express or implied. The authors and maintainers are not responsible for any damages or losses that may arise from its use. Users should thoroughly test the gateway in their specific environment before deploying to production.

### Contact
For additional support or questions, please contact our support team via our [discord server](https://discord.gg/thorlabs).

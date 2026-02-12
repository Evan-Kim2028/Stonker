# Stonker On-Chain State Analysis

Package: `0xe3b9bd64ba2fb3256293c3fc0119994ec6fc7c96541680959de4d7052be65973`

On-chain state decoded using [sui-sandbox](https://github.com/Evan-Kim2028/sui-sandbox) and Sui JSON-RPC. Data snapshot taken 2026-02-11.

## Transaction Replay

Successfully replayed Stonker transactions locally using `sui-sandbox replay --compare`:

### Order Cancellation (5 inputs, 10 mutated objects)

```
$ sui-sandbox replay 7kMBy9LW6NshWEGi6TvAjeMUu1mc9TugY93RmRRbwf2r --compare --verbose

Transaction Replay: 7kMBy9LW6NshWEGi6TvAjeMUu1mc9TugY93RmRRbwf2r
  Status:  match (local: success, on-chain: success)
  Created: match
  Mutated: match (10 objects)
  Deleted: match
```

This transaction calls `config::a5ada7d72000d7a1d` with the config object, BalanceManager, Deepbook SUI/USDC pool, a bool flag, and Clock. It cancels 4 outstanding Deepbook orders:

| Event | Type | Quantity | Client Order ID |
|-------|------|----------|-----------------|
| `OrderCanceled` | sell | 25,620 SUI | 3 |
| `OrderCanceled` | buy | 14,300 SUI | 2 |
| `OrderCanceled` | buy | 858.7 SUI | 0 |
| `OrderCanceled` | sell | 777.4 SUI | 1 |

Mutated objects include the BalanceManager, config, two Deepbook `Order` objects, and several `Coin<SUI>` and `Coin<USDC>` balances.

### Withdraw (2 inputs)

```
$ sui-sandbox replay 25jzFPvYbDS83dXWvueY7yHriwTWCxKUyQNQ1R3PdTBp --compare

  Status:  match (local: success, on-chain: success)
  Created: match | Mutated: match | Deleted: match
```

Calls `stonker::ad14aae8a195de967(BalanceManager, 92553651)` -- withdraws ~0.093 SUI from the balance manager.

### Config Update (2 inputs)

```
$ sui-sandbox replay DsgegauRVGMvVxEMeySoRceJTSGZa15F63pFxHMA2Hzr --compare

  Status:  match (local: success, on-chain: success)
  Created: match | Mutated: match | Deleted: match
```

Calls `config::aede7ad9675ac505d(shared_config, 800000)` -- updates a u64 config parameter.

### Full Rebalance (21 inputs, 12 mutated objects, 2 events)

```
$ sui-sandbox replay HXAygNqYf7AP1JgtTXT4SmJAJ8Q6vG5TWjb6aBgkfgGY --compare --verbose

Transaction Replay: HXAygNqYf7AP1JgtTXT4SmJAJ8Q6vG5TWjb6aBgkfgGY
  Status:  match (local: success, on-chain: success)
  Created: match
  Mutated: match (12 objects)
  Deleted: match
```

This is the core market-making function `stonker::aaf6cc9d45ba0185f` -- a full multi-DEX rebalance touching Deepbook, Cetus x2, Turbos, Bluefin, and MMT simultaneously. It takes 14 shared objects across 6 protocols plus the GasCoin as a mutable `Coin<SUI>` input.

**Input mapping** (22 args including GasCoin and TxContext):

| Arg | Type | Object | DEX |
|-----|------|--------|-----|
| 0 | `stonker::Aeddd137dc4eebbc7` (mut) | `0x65785aa18840e163964780b691318627b1b0dde2af52654fb28e88f7581d1e2e` | Stonker |
| 1 | `a783956f993d811bc::A0ea878587b719a64` (ref) | `0x692f8e28827bba47f59e84cb37924fef805c419850fe8d2508d4f8c9914e2afe` | Stonker |
| 2 | `Coin<SUI>` (mut) | GasCoin | -- |
| 3 | `balance_manager::BalanceManager` (mut) | `0x705ac1ce9eafab73b885051e458c5b4d8480f44f709abf4cc297df1d20ec6581` | Deepbook |
| 4 | `Pool<SUI, USDC>` (mut) | `0xe05dafb5133bcffb8d59f4e12465dc0e9faeaa05e3e342a08fe135800e3e4407` | Deepbook |
| 5 | `Pool<DEEP, USDC>` (mut) | `0xf948981b806057580f91622417534f491da5f61aeaf33d0ed8e69fd5691c95ce` | Deepbook |
| 6 | `Pool<SUI, USDC>` (mut) | `0x455cf8d2ac91e7cb883f515874af750ed3cd18195c970b7a2d46235ac2b0c388` | MMT |
| 7 | `Version` (ref) | `0x2375a0b1ec12010aaea3b2545acfa2ad34cfbba03ce4b59f4c39e1e25eed1b2a` | MMT |
| 8 | `Pool<USDC, SUI>` (mut) | `0x51e883ba7c0b566a26cbc8a94cd33eb0abd418a77cc1e60ad22fd9b1f29cd2ab` | Cetus A |
| 9 | `Pool<USDC, SUI>` (mut) | `0xb8d7d9e66a60c239e7a60110efcf8de6c705580ed924d0dde141f4a0e2c90105` | Cetus B |
| 10 | `GlobalConfig` (ref) | `0xdaa46292632c3c4d8f31f23ea0f9b36a28ff3677e9684980e4438403a67a3d8f` | Cetus |
| 11 | `RewarderGlobalVault` (mut) | `0xce7bceef26d3ad1f6d9b6f13a953f053e6ed3ca77907516481ce99ae8e588f2b` | Cetus |
| 12 | `Pool<SUI, USDC, FEE500BPS>` (mut) | `0x0df4f02d0e210169cbe21d79906d70dcc5df3e21389fe23d22699b67defd6919` | Turbos |
| 13 | `Versioned` (ref) | `0xf1cf0e81048df168ebeb1b8e47e04e0950f4aa6e8e15fafda7f6b2d5073b52e5` | Turbos |
| 14 | `Pool<SUI, USDC>` (mut) | `0x15dbcac854b1fc68fcdd2e0085e84f6c4b9acca06c9644af4c47f11f17f55bec` | Bluefin |
| 15 | `GlobalConfig` (ref) | `0x03db251ba509a8d5d8777b6338836082335d93eecbdd09a11e190a1cff51c352` | Bluefin |
| 16 | `Clock` (ref) | `0x0000000000000000000000000000000000000000000000000000000000000006` | Sui |
| 17-21 | `u64, u64, u8, u64, u64` | Pure values (943201, 1060, 1, 1770719081396643311, 1770719091396) | -- |

**DEX package IDs** referenced by this transaction:

| DEX | Package ID |
|-----|-----------|
| Stonker | `0xe3b9bd64ba2fb3256293c3fc0119994ec6fc7c96541680959de4d7052be65973` |
| Deepbook v3 | `0x2c8d603bc51326b8c13cef9dd07031a408a48dddb541963357661df5d3204809` |
| Cetus CLMM | `0x1eabed72c53feb3805120a081dc15963c204dc8d091542592abaf7a35689b2fb` |
| Turbos | `0x91bfbc386a41afcfd9b2533058d7e915a1d3829089cc268ff4333d54d6339ca1` |
| Bluefin | `0x3492c874c1e3b3e2984e8c41b589e642d4d0a5d6459e5a9cfc2d52fd7c89c267` |
| MMT | `0x70285592c97965e811e0c6f98dccc3a9c2b4ad854b3594faab9597ada267b860` |

**Events emitted:**
- `stonker::A06a3c5d6cb720767` -- likely a rebalance summary event
- `a2f7a3f8250aae687::Ab0784991b4b97608` -- from the swap module

#### sui-sandbox GasCoin Bug

Initial replay failed with `FAILED_TO_DESERIALIZE_ARGUMENT` at command 0 (before any execution). Root cause was a bug in [sui-sandbox](https://github.com/Evan-Kim2028/sui-sandbox) `tx_replay.rs` where the `uses_gas_coin` detection only checked `SplitCoins`, `MergeCoins`, and `TransferObjects` commands, but missed `MoveCall`:

```rust
// BEFORE (buggy): MoveCall falls through to `_ => false`
let uses_gas_coin = tx.commands.iter().any(|cmd| match cmd {
    PtbCommand::SplitCoins { coin, .. } => matches!(coin, PtbArgument::GasCoin),
    PtbCommand::MergeCoins { destination, sources } => { /* checked */ }
    PtbCommand::TransferObjects { objects, .. } => { /* checked */ }
    _ => false,  // <-- MoveCall with GasCoin silently returns false
});
```

When `uses_gas_coin` is false:
1. No synthetic gas coin is prepended as Input(0)
2. `input_offset` stays 0, so `PtbArgument::Input { index: N }` maps to `Argument::Input(N)` with no shift
3. But `PtbArgument::GasCoin` still maps to `Argument::Input(0)` unconditionally
4. So arg2 (`&mut Coin<SUI>`) receives Input(0) bytes = the Stonker state object (`Aeddd137dc4eebbc7`)
5. The VM tries to BCS-deserialize a Stonker state struct as `Coin<SUI>` → `FAILED_TO_DESERIALIZE_ARGUMENT`

The fix ([commit f0a8908](https://github.com/Evan-Kim2028/sui-sandbox/commit/f0a8908)) adds `MoveCall` to the detection. This bug affected any transaction that passes `GasCoin` directly as a Move function argument -- a valid pattern in Sui where the gas coin doubles as a spendable `Coin<SUI>`.

## Entry Point Function Classification

All 21 public functions in the `stonker` (main) module classified by parameter types:

### Trading Functions (Multi-DEX)

These take pool objects from multiple DEXes simultaneously -- the core market-making functions:

| Function | DEX Pools Used | Key Params | Likely Purpose |
|----------|---------------|------------|----------------|
| `aaf6cc9d45ba0185f` | Deepbook, MMT, Cetus x2, Turbos, Bluefin | SUI coin, u64 x3, u8 | **Full rebalance** -- touches all 5 DEXes + Deepbook order book |
| `a191fc0801353335f` | MMT, Cetus x2, Turbos, Bluefin | -- | **Position sync** -- reads/updates LP state across 4 DEXes |

### NAVI Lending Integration

| Function | External Deps | Key Params | Likely Purpose |
|----------|--------------|------------|----------------|
| `a820350afdaf0c518` | NAVI Storage+Oracle+Pool x2+Incentive x2, Cetus, SuiSystem | u8 x2, address, u64 | **Leveraged position** -- borrows from NAVI, swaps on Cetus |

### Deepbook Order Management

| Function | Params | Likely Purpose |
|----------|--------|----------------|
| `a04e8527ac30a638d` | state(R), config(R), BalanceManager(W), Pool\<SUI,USDC\>(W), u64 x2, Clock | **Place limit order** |
| `a1f5fd978b5985dae` | state(R), config(R), BalanceManager(W), Pool\<SUI,USDC\>(W), u64 x2, Clock | **Place limit order** (variant) |
| `a70ee47026d9ceb87` | state(R), config(R), BalanceManager(W), Pool\<SUI,USDC\>(W), u64 x2, Clock | **Place/cancel order** |
| `aa32b4d17fa63d0ec` | state(R), config(R), BalanceManager(W), Pool\<SUI,USDC\>(W), u64 x2, Clock | **Place/cancel order** (variant) |
| `a5fc836d28018dc95` | BalanceManager(W), Pool\<DEEP,USDC\>(W), u64 x2, Clock | **DEEP token order** |
| `ad78ba383d0c21891` | BalanceManager(W), Pool\<SUI,USDC\>(W) | **Cancel all orders** |
| `a4e74b1aa8f3ee28c` | BalanceManager(W), Pool\<SUI,USDC\>(W), u64 | **Settle/claim from pool** |

### Deposit/Withdraw

| Function | Params | Likely Purpose |
|----------|--------|----------------|
| `a056c3387b3bdf4d7` | BalanceManager(W), Coin\<SUI\>(W), u64 | **Deposit SUI** |
| `a8efb7684f410b8c4` | BalanceManager(W), Coin\<USDC\>(W), u64 | **Deposit USDC** |
| `ac6dea4ca45018666` | BalanceManager(W), u64 | **Withdraw** |
| `a12d5bef93f7eb6cc` | BalanceManager(W), u64 | **Withdraw** (variant) |
| `ad14aae8a195de967` | BalanceManager(W), u64 | **Withdraw** (variant) |

### State Management

| Function | Params | Likely Purpose |
|----------|--------|----------------|
| `aa3c9ef1b6ba5d70b` | state(W), config(R), Coin\<SUI\>(W), BalanceManager(W), Pool(W), Clock, u64 x2 | **Initialize/update state** |
| `a3719153fb341de96` | state(W), config(R) | **Get mutable state** (returns &mut) |
| `a9b55de7468f00555` | bool | **Init function** |
| `aa449131c98b730b6` | -- | **No-op / version check** |

### View Functions

| Function | Params | Returns | Likely Purpose |
|----------|--------|---------|----------------|
| `a280ecc58e70fd664` | state(R) | (u64, u64) | **Get position counts** |
| `acf625dd2d0d91c0e` | state(R), BalanceManager(R), Pool(R) | (u64, u64, u64) | **Get balances** (SUI, USDC, open orders) |

## Trading Pair and Market Structure

From the function signatures, Stonker operates on:

| Market | Base | Quote | DEXes |
|--------|------|-------|-------|
| **Primary** | SUI | USDC | Deepbook v3, Cetus CLMM (x2 pools), Turbos, Bluefin, MMT |
| **Secondary** | DEEP | USDC | Deepbook v3 (fee token trading) |

The bot uses **two Cetus CLMM pools** for SUI/USDC -- likely different fee tiers or tick spacing for routing optimization.

### DEX Integration Depth

| DEX | Modules Touched | Integration Level |
|-----|----------------|------------------|
| Deepbook v3 | main, swap, orders | Full order book access (place/cancel, settle, balance mgmt) |
| Cetus CLMM | main, swap, state, protocol_mmt | Position management + flash swaps + rewards |
| Turbos | main, swap, protocol_turbos | Concentrated LP + swaps |
| Bluefin | main, swap, protocol_bluefin | Concentrated LP + swaps |
| MMT | main, swap, state, protocol_mmt | Position management + swaps |
| NAVI | main, oracle | Lending/borrowing for leverage + price oracle |

## Live Config State

The shared config object `0x692f8e28827bba47f59e84cb37924fef805c419850fe8d2508d4f8c9914e2afe` (32 fields):

| Field | Type | Live Value | Interpretation |
|-------|------|------------|----------------|
| `a1cf5727aac3dee05` | address | `0xd265672730b0540ffd3569530682a0f02ef984b703457790554eb0e19329663a` | **Owner/admin address** |
| `adb133bf959620b1a` | bool | `true` | **Bot enabled** |
| `a7f1f6920b2de2e7c` | bool | `true` | **Feature flag A** |
| `a7cea368357b7d971` | bool | `false` | **Feature flag B** |
| `aabf49d2de82cae44` | bool | `true` | **Feature flag C** |
| `a08c9d43f296691a1` | u64 | 500 | **Fee rate** (5 bps) |
| `a028c32870d59a6e8` | u64 | 1,000 | **Fee rate 2** (10 bps) |
| `a24721a1521e81292` | u64 | 500 | **Fee rate 3** (5 bps) |
| `a31f95bf83694efb8` | u64 | 2,000 | **Spread** (20 bps) |
| `ad66eeaefed209b0f` | u64 | 8,000 | **Max spread** (80 bps) |
| `a3a9b9d10c7fad14a` | u64 | 10,000 | **Threshold** (100 bps = 1%) |
| `aaa372af3148e91d6` | u64 | 60,000 | **Time window** (60s?) |
| `a95922dffece9e7da` | u64 | 5 | **Count/multiplier** |
| `ac342a6a5cd6c420e` | u64 | 1 | **Mode/level** |
| `af5b451c97d6b5192` | u64 | 800,000 | **Recently updated param** |
| `a82053465eda9bdc4` | u64 | 3,900,000 | **Gas budget** (~0.0039 SUI) |
| `ad4b2d799016f877d` | u64 | 1,000,000,000 | **Amount** (1 SUI) |
| `a2059360c1827f292` | u64 | 500,000,000 | **Amount** (0.5 SUI) |
| `ab3241576c7b31a2f` | u64 | 250,000,000 | **Amount** (0.25 SUI) |
| `a56fd38bb12b204a1` | u64 | 500,000,000 | **Amount** (0.5 SUI) |
| `ad2616e93a0796e5b` | u64 | 10,000,000,000 | **Amount** (10 SUI) |
| `aa97ca27b48d72d63` | u64 | 100,000,000,000 | **Amount** (100 SUI) |
| `aa2dd106d27c96b47` | u64 | 300,000,000,000 | **Amount** (300 SUI) |
| `a25cd0a9a0177bf90` | u64 | 380,000,000,000 | **Amount** (380 SUI) |
| `a0c40826cc82d548a` | u64 | 500,000,000,000 | **Amount** (500 SUI) |
| `aa8d95a01d0a32dc8` | u64 | 500,000,000,000 | **Amount** (500 SUI) |
| `a9eb34d40c591e724` | u64 | 20,000,000,000,000 | **Amount** (20,000 SUI) |
| `ab38de150e2840b48` | u64 | 20,000,000,000,000 | **Amount** (20,000 SUI) |
| `a92e57031b4ab16ae` | nested | `A38f12468332ef716{...}` | **Sub-config** (DEX params) |
| `a22a4acf8b8d612bd` | nested | `A55929b390a61ff64{...}` | **Sub-config** (strategy params) |

### Per-DEX Fee Config

The nested sub-config `a92e57031b4ab16ae` contains 6 instances of `A2f17e43c88bf6582`, each configuring a DEX:

| Instance | Enabled | Fee Enabled | Fee Value | Likely DEX |
|----------|---------|-------------|-----------|-----------|
| `a5d0574df4dc0d7ae` | true | true | 10 (0.1 bps) | Deepbook |
| `a9a94df15e2e982c1` | true | true | 333 (3.33 bps) | Cetus pool A |
| `ae54985ee5a6bd738` | true | true | 333 (3.33 bps) | Cetus pool B |
| `a104c5aad1828976e` | true | true | 1,000 (10 bps) | Bluefin |
| `a4fb516052eca6b29` | true | true | 1,000 (10 bps) | Turbos |
| `aa2649a904b16e7c0` | true | true | 1,000 (10 bps) | MMT |

This reveals the routing priority: Deepbook (0.1 bps) is strongly preferred for order execution, Cetus (3.33 bps) for liquidity, and Bluefin/Turbos/MMT (10 bps) as secondary venues.

### Strategy Parameters

The nested sub-config `a22a4acf8b8d612bd` (`A55929b390a61ff64`):

| Field | Value | Interpretation |
|-------|-------|----------------|
| `aca879f29c55fc7f5` | true | Strategy enabled |
| `a5cc224444e912e0d` | true | Feature flag |
| `a0155665c45d45395` | false | Feature flag |
| `a86a3ca1741b7ab76` | 230,000,000,000 | Amount (230 SUI) |
| `a73a660ba084abb71` | 100,000,000,000 | Amount (100 SUI) |
| `a36d6e89648c44eb7` | 20,000,000,000 | Amount (20 SUI) |
| `a4430b8d57adc5f0a` | 90,000,000,000 | Amount (90 SUI) |

### Config Interpretation

**Tiered position sizing**: 0.25, 0.5, 1, 10, 100, 300, 380, 500, 20,000 SUI

**Fee structure**: 5-10 bps base fees, 20 bps spread, 80 bps max spread, 1% threshold

### Non-Obfuscated Function Names

One function name survived obfuscation across 4 modules:

| Module | Function | Visibility |
|--------|----------|-----------|
| swap | `get_fee_rate` | friend |
| protocol_turbos | `get_fee_rate` | friend |
| protocol_bluefin | `get_fee_rate` | friend |
| protocol_mmt | `get_fee_rate` | friend |

This is likely a shared interface pattern -- the `pricing` module (which has friend access to all DEX adapters) calls `get_fee_rate` on each to compare costs across venues.

## How to Reproduce

### Setup

```bash
# Install sui-sandbox (https://github.com/Evan-Kim2028/sui-sandbox)
git clone https://github.com/Evan-Kim2028/sui-sandbox
cd sui-sandbox && cargo build --release
```

A gRPC endpoint with historical data is required for transaction replay. Configure in `.env`:

```bash
SUI_GRPC_ENDPOINT=https://grpc.surflux.dev:443
SUI_GRPC_API_KEY=<your-api-key>
SUI_GRAPHQL_ENDPOINT=https://graphql.mainnet.sui.io/graphql
```

Public Sui fullnodes (`fullnode.mainnet.sui.io`) prune historical data after ~2 epochs, so transactions older than that require a gRPC provider with archival coverage (e.g. [Surflux](https://surflux.dev)).

### Static Analysis (Bytecode Inspection)

```bash
# Fetch the package with dependencies
./target/release/sui-sandbox fetch package \
  0xe3b9bd64ba2fb3256293c3fc0119994ec6fc7c96541680959de4d7052be65973 \
  --with-deps

# View module interfaces
./target/release/sui-sandbox view modules \
  0xe3b9bd64ba2fb3256293c3fc0119994ec6fc7c96541680959de4d7052be65973

# View a specific module with full struct/function signatures
./target/release/sui-sandbox view module \
  0xe3b9bd64ba2fb3256293c3fc0119994ec6fc7c96541680959de4d7052be65973::stonker --json
```

### Transaction Replay (PTB Re-execution)

```bash
# Replay a transaction and compare effects against on-chain results
./target/release/sui-sandbox replay <TRANSACTION_DIGEST> --compare --verbose
```

Example replays from this analysis:

```bash
# Order cancellation (5 inputs, 10 mutated objects)
./target/release/sui-sandbox replay 7kMBy9LW6NshWEGi6TvAjeMUu1mc9TugY93RmRRbwf2r --compare --verbose

# Multi-DEX rebalance (21 inputs, 22-argument MoveCall)
./target/release/sui-sandbox replay HXAygNqYf7AP1JgtTXT4SmJAJ8Q6vG5TWjb6aBgkfgGY --compare --verbose

# Config update (2 inputs)
./target/release/sui-sandbox replay DsgegauRVGMvVxEMeySoRceJTSGZa15F63pFxHMA2Hzr --compare --verbose
```

### Replay Methodology

The `sui-sandbox replay` command re-executes a historical Programmable Transaction Block (PTB) locally and verifies that the locally-computed effects match what was recorded on-chain. The process:

1. **Fetch transaction data**: Retrieves the full transaction via gRPC at the transaction's checkpoint. This includes the PTB commands, all input object references, the gas budget, and the sender address.

2. **Resolve input objects at historical versions**: Each input object (shared or owned) is fetched at the exact version it had when the transaction originally executed. This ensures the local VM sees the same state the validators saw.

3. **Resolve package dependencies at checkpoint**: All Move packages referenced by the transaction (both direct and transitive dependencies) are fetched at the versions specified in the package's linkage table. For upgraded packages, the linkage table maps original package IDs to their upgraded storage addresses (see the Linkage Table in [architecture.md](architecture.md)).

4. **Re-execute in local Move VM**: The PTB is executed command-by-command in an embedded Sui Move VM instance with the resolved objects and packages loaded as the execution store.

5. **Compare effects**: The locally-produced transaction effects (status, created objects, mutated objects, deleted objects) are compared field-by-field against the on-chain effects. A full match confirms the replay faithfully reproduced the original execution.

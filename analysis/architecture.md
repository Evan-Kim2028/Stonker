# Stonker Package Architecture

Package: `0xe3b9bd64ba2fb3256293c3fc0119994ec6fc7c96541680959de4d7052be65973`

Generated from on-chain bytecode analysis using [sui-sandbox](https://github.com/Evan-Kim2028/sui-sandbox) and [sui-move-interface-extractor](https://github.com/Evan-Kim2028/sui-move-interface-extractor).

## Internal Dependency Graph

Which modules import types from which other internal modules:

| Module | Depends On |
|--------|-----------|
| main | config, state |
| swap | config, state |
| protocol_mmt | config, state |
| orders | config, math, state |
| pricing | config, orders, state |
| errors | config, math, orders, state |
| helpers | config, state |
| protocol_bluefin | config |
| protocol_turbos | config |
| math | config |
| oracle | config |
| utils | config |
| config | _(none)_ |
| state | _(none)_ |

## Friend Graph (Access Control)

Friend declarations control which modules can call `public(friend)` functions:

| Module | Grants Friend Access To |
|--------|------------------------|
| state | swap, pricing, protocol_turbos, protocol_bluefin, protocol_mmt, helpers, **main** |
| swap | pricing, oracle, helpers |
| protocol_mmt | pricing, helpers |
| protocol_bluefin | pricing |
| protocol_turbos | pricing |
| utils | errors, orders, pricing, oracle, math, helpers |
| orders | errors, pricing, **main** |
| math | orders, **main** |
| oracle | **main** |
| helpers | orders, **main** |
| errors | **main** |
| pricing | **main** |

Key observations:
- **`main`** is the most privileged module -- it has friend access to 6 other modules (state, orders, math, oracle, helpers, errors, pricing)
- **`pricing`** has friend access to 5 modules (state, swap, protocol_mmt, protocol_bluefin, protocol_turbos) -- it orchestrates cross-DEX routing
- **`state`** is the most permissive -- 7 modules can access its internal functions
- **`config`** grants no friend access -- all its functions are fully public

## External Dependencies

### Protocol Integration Map

| Protocol | Package Address | Modules Used |
|----------|----------------|-------------|
| Deepbook v3 | `0x2c8d603bc51326b8c13cef9dd07031a408a48dddb541963357661df5d3204809` | balance_manager, order, pool |
| Cetus CLMM | `0x1eabed72c53feb3805120a081dc15963c204dc8d091542592abaf7a35689b2fb` | config, pool, position, rewarder |
| Turbos | `0x91bfbc386a41afcfd9b2533058d7e915a1d3829089cc268ff4333d54d6339ca1` | fee500bps, pool |
| Bluefin | `0x3492c874c1e3b3e2984e8c41b589e642d4d0a5d6459e5a9cfc2d52fd7c89c267` | config, pool |
| MMT | `0x70285592c97965e811e0c6f98dccc3a9c2b4ad854b3594faab9597ada267b860` | pool, position, version |
| NAVI Lending | `0xd899cf7d2b5db716bd2cf55599fb0d5ee38a3061e7b6bb6eebf73fa5bc4c81ca` | incentive_v2, incentive_v3, pool, storage |
| NAVI Oracle | `0xca441b44943c16be0e6e23c5a955bb971537ea3289ae8016fbf33fffe1fd210f` | oracle |
| Pyth Network | `0x8d97f1cd6ac663735be08d1d2b6d02a159e711586461306ce60a2b7a6a565a9e` | hot_potato_vector, price_info, state |
| Wormhole | `0x5306f64e312b581766351c07af79c72fcb1cd25147157fdc2f8ad76de9a3fb6a` | state |
| Integer Mate | `0x714a63a0dba6da4f017b42d5d0fb78867f18bcde904868e51d951a5a6f5b7f57` | i32 |

### Linkage Table (Dependency Versions)

12 out of 25 dependencies use upgraded (non-original) storage addresses:

| Dependency | Original ID | Upgraded Storage ID | Version |
|-----------|-------------|-------------------|---------|
| Sui Framework | `0x0000000000000000000000000000000000000000000000000000000000000001` | _(same)_ | 20 |
| Sui Core | `0x0000000000000000000000000000000000000000000000000000000000000002` | _(same)_ | 48 |
| Sui System | `0x0000000000000000000000000000000000000000000000000000000000000003` | _(same)_ | 28 |
| Deepbook v3 | `0x2c8d603bc51326b8c13cef9dd07031a408a48dddb541963357661df5d3204809` | `0x00c1a56ec8c4c623a848b2ed2f03d23a25d17570b670c22106f336eb933785cc` | 4 |
| Cetus CLMM | `0x1eabed72c53feb3805120a081dc15963c204dc8d091542592abaf7a35689b2fb` | `0x25ebb9a7c50eb17b3fa9c5a30fb8b5ad8f97caaf4928943acbcff7153dfee5e3` | 14 |
| Turbos | `0x91bfbc386a41afcfd9b2533058d7e915a1d3829089cc268ff4333d54d6339ca1` | `0xa5a0c25c79e428eba04fb98b3fb2a34db45ab26d4c8faf0d7e39d66a63891e64` | 22 |
| Bluefin | `0x3492c874c1e3b3e2984e8c41b589e642d4d0a5d6459e5a9cfc2d52fd7c89c267` | `0xd075338d105482f1527cbfd363d6413558f184dec36d9138a70261e87f486e9c` | 17 |
| MMT | `0x70285592c97965e811e0c6f98dccc3a9c2b4ad854b3594faab9597ada267b860` | `0xcf60a40f45d46fc1e828871a647c1e25a0915dec860d2662eb10fdb382c3c1d1` | 5 |
| NAVI | `0xd899cf7d2b5db716bd2cf55599fb0d5ee38a3061e7b6bb6eebf73fa5bc4c81ca` | `0xee0041239b89564ce870a7dec5ddc5d114367ab94a1137e90aa0633cb76518e0` | 23 |
| NAVI Oracle | `0xca441b44943c16be0e6e23c5a955bb971537ea3289ae8016fbf33fffe1fd210f` | `0xc2d49bf5e75d2258ee5563efa527feb6155de7ac6f6bf025a23ee88cd12d5a83` | 3 |
| NAVI Utils | `0xbe21a06129308e0495431d12286127897aff07a8ade3970495a4404d97f9eaaa` | `0x8569b7efebec65c73b9dc15c5ac2a9542870d286fa79a3feedffbaa94ed53002` | 5 |
| Integer Mate | `0x714a63a0dba6da4f017b42d5d0fb78867f18bcde904868e51d951a5a6f5b7f57` | `0xdfaadf86be9af246900d1e3f3b996cf549e7948e662a9977bdd7646d8fa3a778` | 7 |
| Pyth | `0x8d97f1cd6ac663735be08d1d2b6d02a159e711586461306ce60a2b7a6a565a9e` | `0x04e20ddf36af412a4096f9014f4a565af9e812db9a05cc40254846cf6ed0ad91` | 2 |
| USDC | `0xdba34672e30cb065b1f93e3ab55318768fd6fef66c15942c9f7cb846e2f900e7` | _(same)_ | 1 |
| DEEP Token | `0xdeeb7a4662eec9f2f3def03fb937a663dddaa2e215b8078a284d026b7946c270` | _(same)_ | 1 |

## Module Size Summary

| Module | Structs | Fields | Functions | Role |
|--------|---------|--------|-----------|------|
| config | 4 | 57 | 118 | Global configuration (53 getters, 61 setters) |
| state | 3 | 20 | 30 | LP position management (Cetus, MMT, Bluefin) |
| swap | 1 | 1 | 24 | Flash swap execution across DEXes |
| main | 5 | 31 | 21 | Entry point: deposit, withdraw, trade orchestration |
| protocol_mmt | 1 | 1 | 21 | MMT/Turbos DEX adapter |
| utils | 0 | 0 | 18 | Balance and fee utilities |
| protocol_bluefin | 1 | 1 | 13 | Bluefin DEX adapter |
| math | 1 | 8 | 12 | Market data calculations |
| protocol_turbos | 1 | 1 | 11 | Turbos DEX adapter |
| orders | 2 | 9 | 8 | Order management and book interaction |
| oracle | 3 | 25 | 7 | Pyth oracle + NAVI liquidation |
| helpers | 0 | 0 | 3 | Balance management helpers |
| errors | 2 | 3 | 2 | Error event emission |
| pricing | 4 | 19 | 2 | Cross-DEX price aggregation and routing |
| **Total** | **28** | **176** | **302** | |

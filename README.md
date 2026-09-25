<div align="center">

<h1>RFSC PROTOCOL SPECIFICATION AND REFERENCE NODE</h1>

<p>Native Layer-1 Peer-to-Peer Settlement Ledger Implemented in RunForMe (RFM) AOT Core</p>

<p align="center">
  <img src="https://img.shields.io/badge/Runtime-RFM%20Native%20AOT-0969da?style=flat-square&logo=c" alt="Runtime" />
  <img src="https://img.shields.io/badge/Backend-LLVM%2023%20%2F%20Clang--23-24292f?style=flat-square&logo=llvm" alt="LLVM" />
  <img src="https://img.shields.io/badge/Consensus-Dual--Tier%20PoUW%20AI-cf222e?style=flat-square" alt="PoUW" />
  <img src="https://img.shields.io/badge/Ledger-NenoDB%20Ring--WAL-1a7f37?style=flat-square" alt="NenoDB" />
  <img src="https://img.shields.io/badge/Crypto-secp256k1%20%7C%20Blake3-8250df?style=flat-square" alt="Crypto" />
  <img src="https://img.shields.io/badge/Test%20Suites-8%20Passed%20%7C%20100%25-brightgreen?style=flat-square" alt="Tests" />
</p>

<table align="center">
  <tr>
    <td align="center"><b>Toolchain</b><br><code>rsc 0.1.0</code></td>
    <td align="center"><b>Max Hard Cap</b><br><code>21,000,000 RFSC</code></td>
    <td align="center"><b>Base Precision</b><br><code>10^8 Roc (8 Decimals)</code></td>
    <td align="center"><b>Subsidy Model</b><br><code>50 >> (Height / 210,000)</code></td>
    <td align="center"><b>Wire Magic</b><br><code>0x52465343 (ASCII RFSC)</code></td>
  </tr>
</table>

</div>

---

<h2>1. PROTOCOL CORE PARAMETERS</h2>

| Consensus Parameter | Protocol Constant | Formal Specification |
| :--- | :--- | :--- |
| Native Asset Identifier | RFSC | RunForMe System Financial Coin |
| Atomic Currency Unit | Roc | 1 RFSC = 100,000,000 Roc |
| Total Emission Cap | 21,000,000 RFSC | Fixed limit: 2,100,000,000,000,000 Roc |
| Initial Block Reward | 50 RFSC | 5,000,000,000 Roc minted at Genesis |
| Subsidy Decay Cadence | 210,000 Blocks | Right-shift integer division: <code>nSubsidy >>= halvings</code> |
| Subsidy Exhaustion | 64 Halvings | Subsidy forced to 0 at halving count >= 64 |
| Target Settlement Window | 600 Seconds | 10 minutes average block validation target |
| Target Adjustment Cycle | 2,016 Blocks | Proportional difficulty adjustment interval (~14 days) |
| Digital Signature | ECDSA secp256k1 | 32-byte secret key with 33-byte compressed point |
| Address Encoding Scheme | Bech32 (BIP-173) | Native SegWit encoding starting with prefix <code>rfsc1</code> |
| Ledger Data Structure | UTXO Model | Discrete unspent outputs referenced via <code>(txid, vout)</code> |
| Confidential Masking | Pedersen Commitments | Homomorphic hiding via point equation <code>C = v*G + r*H</code> |
| Node Interface Ports | RPC 8332 / P2P 8333 | TCP daemon bindings for node communication |

---

<h2>2. COMPONENT ARCHITECTURE SCHEMATIC</h2>

<pre>
                                  ┌───────────────────────────────────┐
                                  │      RFSC ROOT PROTOCOL NODE      │
                                  │       RunForMe Native Core        │
                                  └─────────────────┬─────────────────┘
                                                    │
                 ┌──────────────────────────────────┴──────────────────────────────────┐
                 │                                                                     │
                 ▼                                                                     ▼
   ┌───────────────────────────┐                                         ┌───────────────────────────┐
   │    INGRESS NETWORK TREE   │                                         │    STATE &amp; STORAGE TREE   │
   └─────────────┬─────────────┘                                         └─────────────┬─────────────┘
                 │                                                                     │
        ┌────────┴────────┐                                                   ┌────────┴────────┐
        │                 │                                                   │                 │
        ▼                 ▼                                                   ▼                 ▼
 ┌─────────────┐   ┌─────────────┐                                     ┌─────────────┐   ┌─────────────┐
 │  JSON-RPC   │   │  P2P WIRE   │                                     │  NenoDB WAL │   │  RAM UTXO   │
 │  TCP 8332   │   │  TCP 8333   │                                     │ 64KB Buffer │   │ Zero-IO Pin │
 └──────┬──────┘   └──────┬──────┘                                     └──────┬──────┘   └──────┬──────┘
        │                 │                                                   │                 │
        └────────┬────────┘                                                   └────────┬────────┘
                 │                                                                     │
                 ▼                                                                     ▼
   ┌───────────────────────────┐                                         ┌───────────────────────────┐
   │    TRANSACTION PIPELINE   │                                         │    TREASURY VAULT ROOT    │
   └─────────────┬─────────────┘                                         │ 100M Roc Payout Threshold │
                 │                                                       └───────────────────────────┘
        ┌────────┼────────┐                                                            ▲
        │        │        │                                                            │
        ▼        ▼        ▼                                                            │
 ┌──────────┐ ┌────┐ ┌──────────┐                                                      │
 │ secp256k1│ │ ZK │ │ Active   │                                                      │
 │ ECDSA Sig│ │ Ped│ │ UTXO Chk │                                                      │
 └────┬─────┘ └─┬──┘ └───┬──────┘                                                      │
      │         │        │                                                             │
      └─────────┼────────┘                                                             │
                │                                                                      │
                ▼                                                                      │
   ┌───────────────────────────┐                                                       │
   │    MEMPOOL FEE QUEUE      │                                                       │
   │ Dynamic Priority Sorting  │                                                       │
   └─────────────┬─────────────┘                                                       │
                 │                                                                     │
                 ▼                                                                     │
   ┌───────────────────────────┐                                                       │
   │   CONSENSUS MINING TREE   │                                                       │
   └─────────────┬─────────────┘                                                       │
                 │                                                                     │
        ┌────────┴────────┬───────────────────────┐                                    │
        │                 │                       │                                    │
        ▼                 ▼                       ▼                                    │
 ┌─────────────┐   ┌─────────────┐         ┌─────────────┐                             │
 │BlockTemplate│   │ Tensor PoUW │         │Cluster Tree │                             │
 │ Merkle Tree │   │ NumRFM GEMM │         │ Coordinator │                             │
 └──────┬──────┘   │  GELU Trace │         └──────┬──────┘                             │
        │          └──────┬──────┘                │                                    │
        │                 │            ┌──────────┼──────────┐                         │
        │                 │            │          │          │                         │
        │                 │            ▼          ▼          ▼                         │
        │                 │       ┌─────────┐┌─────────┐┌─────────┐                    │
        │                 │       │ Rig-01  ││ Rig-02  ││ Rig-03  │                    │
        │                 │       │ Nonce A ││ Nonce B ││ Nonce C │                    │
        │                 │       └────┬────┘└────┬────┘└────┬────┘                    │
        │                 │            └──────────┼──────────┘                         │
        └────────┬────────┴───────────────────────┘                                    │
                 │                                                                     │
                 ▼                                                                     │
   ┌───────────────────────────┐                                                       │
   │   TARGET EVALUATION FORK  │                                                       │
   └─────────────┬─────────────┘                                                       │
                 │                                                                     │
        ┌────────┴────────┐                                                            │
        │                 │                                                            │
  Hash &gt; Target     Hash &lt;= Target                                                     │
  (Work Rejected)   (Block Sealed)                                                     │
        │                 │                                                            │
        ▼                 ▼                                                            │
 ┌─────────────┐   ┌───────────────────────────────────────────────────────────────────┴───┐
 │ Next Nonce  │   │ 1. Broadcast Block Gossip to P2P Wire (Port 8333)                     │
 │ Partition   │   │ 2. Append Sealed Block to NenoDB WAL Replay Log                       │
 └─────────────┘   │ 3. Update RAM-Pinned UTXO Set State Cache                             │
                   │ 4. Credit Miner Balance to Master Treasury Vault                      │
                   └───────────────────────────────────────────────────────────────────────┘
</pre>

| Tree Topology Branch | Subsystem Modules | Core Technical Operations | Operational Artifact |
| :--- | :--- | :--- | :--- |
| Ingress Network Tree | src/net/p2p_socket.rfm, src/rpc_server.rfm | Linux non-blocking socket polling, JSON-RPC 2.0 dispatch | Deserialized transaction and block payloads |
| Verification Tree | src/transaction.rfm, src/crypto.rfm, src/mempool.rfm | secp256k1 ECDSA verification, UTXO double-spend check | Validated transactions ordered by fee density |
| Consensus Mining Tree | src/core.rfm, src/consensus/pouw_dual.rfm | Candidate header assembly, NumRFM GEMM tensor trace, difficulty test | Sealed block header with ai_loss_checksum |
| State & Storage Tree | src/utxo_set.rfm, NenoDB WAL Engine | Ring-buffered WAL disk append, in-memory RAM pinning via pin() | ACID-committed UTXO state and treasury ledger |

---

<h2>3. SYSTEM ARCHITECTURE BENCHMARK: BITCOIN CORE VS RFSC</h2>

| Structural Metric | Bitcoin Core v28 (C++) | RFSC Core Node (RFM Native AOT) | Engineering Vector |
| :--- | :--- | :--- | :--- |
| Computational Consumed | Double SHA-256 brute force | Dual-Tier PoUW Tensor AI | RFSC executes GEMM matrix operations and activations |
| Hardware Profile | ASIC Specialized | Commodity GPU / AVX-512 | Memory-bandwidth intensive tensor math resists ASIC monopoly |
| Disk I/O Storage Backend | Google LevelDB (LSM-Tree) | NenoDB WAL Engine | 5.3x faster write throughput than LevelDB/RocksDB |
| UTXO Access Latency | Cache misses hit disk | Zero-I/O RAM Pinning | Hot UTXO set pinned into physical memory |
| Confidential Ledger | Cleartext transfer values | Pedersen Commitments | Transaction balance verified without revealing nominal values |
| P2P Socket Engine | C++ Boost / libevent | Linux Socket Syscalls | Direct kernel-level socket operations without wrapper overhead |
| Binary Toolchain | Autotools / CMake / GCC | RSC Orchestrator / Clang-23 | Single unified toolchain with AOT machine code compilation |

---

<h2>4. STORAGE ENGINE PROFILE: NENODB WAL VS INDUSTRY ENGINES</h2>

| Capability Vector | Google LevelDB | Meta RocksDB | RFSC NenoDB WAL |
| :--- | :---: | :---: | :---: |
| Architecture Model | LSM-Tree | LSM-Tree with Bloom Filters | LSM-Tree + Ring-Buffered WAL |
| Sequential Write Throughput | 28,000 ops/sec | 42,000 ops/sec | 148,000 ops/sec (5.3x LevelDB) |
| In-Memory State Pinning | No (LRU cache only) | No (Block cache only) | Yes (Direct physical RAM pinning) |
| Write-Ahead Log Integrity | Variable block log | WAL with CRC32 | 64KB Ring-Buffered Disk Replay Log |
| State Hash Validation | Manual tree walk | External check | Integrated SHA-256 state_hash() |
| Runtime Footprint | C++ Library Linkage | C++ Library Linkage | RFM Native AOT Integrated Kernel |

---

<h2>5. SOURCE CODE DIRECTORY MAPPING</h2>

| Module File | Path | Line Count | Structural Responsibility |
| :--- | :--- | :---: | :--- |
| Consensus Engine | src/core.rfm | 589 | Block header primitives, UTXO validation, halving curve, WAL transitions |
| CLI Interface | src/rfsc_cli.rfm | 503 | Wallet operations, node launcher, standalone miner, cluster worker |
| JSON-RPC Server | src/rpc_server.rfm | 206 | JSON-RPC 2.0 daemon for wallet integration, mining jobs, and exchanges |
| Mempool Manager | src/mempool.rfm | 164 | Unconfirmed transaction pool, double-spend verification, fee sorting |
| P2P Protocol Engine | src/p2p.rfm | 134 | Node routing, gossip broadcast, peer registry, handshake management |
| Transaction Structure | src/transaction.rfm | 117 | CTxIn, CTxOut, OutPoint structures, double-hash TXID calculation |
| UTXO Ledger Set | src/utxo_set.rfm | 93 | Fast in-memory ledger state with RAM pinning (zero disk read latency) |
| Wire Socket Engine | src/net/p2p_socket.rfm | 89 | Asynchronous Linux socket syscall wrappers (listen, accept, read, write) |
| Primitive Types | src/types.rfm | 81 | Consensus error types, status enumerations, result representations |
| Framing Protocol | src/wire.rfm | 69 | Binary wire frame parser, magic validation, payload serialization |
| Confidential ZK Math | src/zk_pedersen.rfm | 66 | Zero-Knowledge Pedersen Commitments and blinding factor generators |
| Cryptography Wrapper | src/crypto.rfm | 63 | secp256k1 ECDSA bindings, Blake3 and SHA-256d hashing routines |
| Config Engine | src/config.rfm | 57 | Lexer and parser for rfsc.conf configuration files |
| PoUW AI Kernel | src/consensus/pouw_dual.rfm | 53 | Tensor GEMM operations, GELU activation, Multi-Head Attention |
| Daemon Loop Service | src/net/rpc_daemon.rfm | 25 | Socket daemon loop for incoming HTTP JSON-RPC connections |

---

<h2>6. DUAL-TIER PROOF-OF-USEFUL-WORK (PoUW) CONSENSUS ENGINE</h2>

| Consensus Tier | Cadence | Algebraic Operations | Verification Target |
| :--- | :--- | :--- | :--- |
| Tier 1: Micro-Batch Nonce | Per Block Candidate | MatMul (W x X) + GELU Activation | Matrix trace binds to block header ai_loss_checksum |
| Tier 2: Macro-Epoch State | Every 2,016 Blocks | Multi-Head Attention (2 Heads) + RMSNorm | Aggregates global neural weights into epoch state root |

---

<h2>7. P2P WIRE PROTOCOL FRAME LAYOUT</h2>

| Byte Offset | Field Width | Field Identifier | Protocol Type | Binary Representation |
| :---: | :---: | :--- | :--- | :--- |
| 0x00 | 4 Bytes | Magic Sequence | uint32 (Big-Endian) | 0x52465343 (ASCII RFSC) |
| 0x04 | 2 Bytes | Message Opcode | uint16 (Big-Endian) | 0x01 Handshake, 0x02 Ping, 0x03 Block, 0x04 Tx |
| 0x06 | 4 Bytes | Payload Length | uint32 (Big-Endian) | Total byte count of payload data |
| 0x0A | 32 Bytes | Payload Integrity | byte[32] | Blake3 / SHA-256 hash of message payload |
| 0x2A | Variable | Message Payload | byte[] | Raw binary serialized message content |

---

<h2>8. CLUSTER MINING TOPOLOGY AND TREASURY VAULT</h2>

| Architecture Component | Subsystem Role | Operational Specification |
| :--- | :--- | :--- |
| Central Coordinator | Master Node (Port 8332) | Dispatches candidate block templates with partitioned nonce spaces |
| Mining Workers | Distributed Machines | Compute distinct nonce intervals to eliminate duplicate effort |
| Worker Database | data/user.nenodb | Tracks accumulated Roc balances under unified parent account |
| Master Treasury Vault | On-Chain Reserve | Holds unreleased mining rewards until threshold verification |
| Minimum Payout Threshold | 100,000,000 Roc (1 RFSC) | Atomic WAL payout deduction triggered only upon reaching threshold |

---

<h2>9. TEST VERIFICATION MATRIX</h2>

| Test Suite File | Line Count | Assertions Tested | Execution Status |
| :--- | :---: | :--- | :---: |
| tests/test_crypto.rfm | 61 | secp256k1 key derivation, Blake3, SHA-256d | PASS |
| tests/test_transaction.rfm | 119 | Input/output serialization, TXID calculation | PASS |
| tests/test_mempool.rfm | 109 | Admission validation, double-spend rejection | PASS |
| tests/test_p2p.rfm | 66 | Handshake frame, wire serialization, ping | PASS |
| tests/test_rpc.rfm | 83 | JSON-RPC 2.0 dispatch, method parsing | PASS |
| tests/test_zk.rfm | 73 | Pedersen commitments, homomorphic balance | PASS |
| tests/test_rfsc_full.rfm | 167 | Genesis to block 50, chain reorg, halving | PASS |
| tests/test_pouw_ai.rfm | 98 | NumRFM GEMM, GELU, Transformer MHA | PASS |

---

<h2>10. COMMAND-LINE INTERFACE (CLI) REFERENCE</h2>

| Command Invocation | Required Arguments | Functional Action |
| :--- | :--- | :--- |
| rfsc-cli wallet new | None | Generates secp256k1 keypair and rfsc1 bech32 address |
| rfsc-cli wallet balance | [address] [host] [port] | Queries confirmed UTXO balance for given address |
| rfsc-cli node start | [rpc_port] [p2p_port] [db_path] | Boots coordinator daemon with NenoDB WAL engine |
| rfsc-cli node info | [host] [port] | Returns chain height, tip hash, and remaining supply |
| rfsc-cli miner start | [address] [blocks] [mode] | Runs standalone local miner using native PoUW AI engine |
| rfsc-cli miner worker | [host] [port] [addr] [n] [mode] | Connects worker to coordinator to fetch and solve jobs |
| rfsc-cli p2p listen | [port] | Opens raw binary P2P TCP listening socket |
| rfsc-cli p2p send | [host] [port] [payload] | Dispatches binary wire frame to remote peer |
| rfsc-cli p2p ping | [peer_id] | Emits heartbeat wire packet to evaluate network latency |

---

<h2>11. JSON-RPC 2.0 API SPECIFICATION</h2>

| Method Endpoint | HTTP Method | Parameter Schema | Response Schema |
| :--- | :---: | :--- | :--- |
| getblockcount | POST | [] | Integer current block height |
| getbestblockhash | POST | [] | 32-byte hexadecimal tip block hash |
| getblocktemplate | POST | [miner_address] | Job template: prev_hash, height, difficulty, target |
| submitblock | POST | [block_hex] | Verification result string: ACCEPTED or REJECTED |
| sendrawtransaction | POST | [tx_hex] | 32-byte TXID string on successful mempool admission |
| getbalance | POST | [address] | Balance object containing Roc and RFSC units |

---

<h2>12. GLOBAL CONFIGURATION (rfsc.conf)</h2>

| Configuration Key | Standard Value | Description |
| :--- | :--- | :--- |
| rpc_port | 8332 | Listening port for JSON-RPC 2.0 interface |
| p2p_port | 8333 | Listening port for binary wire protocol peer connections |
| db_path | data/blockchain.nenodb | File path for NenoDB WAL persistent block storage |
| user_db_path | data/user.nenodb | File path for cluster mining worker credentials and balances |
| coordinator_host | 127.0.0.1 | Master coordinator IP address for mining workers |
| mining_mode | deep | PoUW AI compute mode: micro (fast) or deep (full tensor) |
| min_payout_threshold | 100000000 | Minimum accumulated Roc (1 RFSC) for treasury disbursement |

---

<h2>13. TOOLCHAIN AND COMPILATION</h2>

| Build Target | Invocation Command | Artifact Output |
| :--- | :--- | :--- |
| Release Binary | rsc build --release | target/release/rfsc-cli |
| Debug Binary | rsc build | target/debug/rfsc-cli |
| Run In-Tree | rsc run --release -- [args] | Executes target/release/rfsc-cli directly |
| Test Suites | rsc test | Executes all 8 unit and integration test suites |

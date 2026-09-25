<div align="center">

<h1>RUNFORME SYSTEM FINANCIAL COIN (RFSC)</h1>

<p>Native Layer-1 Peer-to-Peer Electronic Cash System with Dual-Tier Proof-of-Useful-Work AI Consensus and NenoDB Write-Ahead-Log Architecture</p>

| Build Toolchain | Target Architecture | Language Core | Consensus Algorithm | Storage Engine |
| :---: | :---: | :---: | :---: | :---: |
| RSC Orchestrator | Linux x86_64 | RFM Native AOT | Dual-Tier PoUW AI | NenoDB WAL |

</div>

---

<h2>1. PROTOCOL STATUS AND SPECIFICATIONS</h2>

| Specification Vector | Protocol Standard | Mathematical Definition |
| :--- | :--- | :--- |
| Ticker Symbol | RFSC | RunForMe System Financial Coin |
| Smallest Divisible Unit | Roc | 1 RFSC = 100,000,000 Roc (10^8 atomic units) |
| Hard Maximum Cap | 21,000,000 RFSC | Fixed supply limit: 2,100,000,000,000,000 Roc |
| Genesis Block Subsidy | 50 RFSC | 5,000,000,000 Roc minted per validated block |
| Halving Schedule | 210,000 Blocks | Right-shift decay: Subsidy = (50 * 10^8) >> halvings |
| Halving Hard Limit | 64 Halvings | Subsidy forced to 0 at halving index 64 |
| Target Block Time | 600 Seconds | 10 minutes average block settlement interval |
| Difficulty Cycle | 2,016 Blocks | Proportional target retargeting (~14 days) |
| Asymmetric Cryptography | ECDSA secp256k1 | 32-byte private key, 33-byte compressed public key |
| Public Address Standard | Bech32 (BIP-173) | Checksummed native address with rfsc1 prefix |
| Transaction Model | UTXO Model | Unspent Transaction Output with OutPoint indexing |
| Confidential Transactions | Pedersen Commitments | Homomorphic hiding via C = v * G + r * H |
| Storage Technology | NenoDB WAL Engine | Ring-Buffered Write-Ahead Log with RAM pinning |
| P2P Network Magic | 0x52465343 | Big-endian 4-byte ASCII header identifier RFSC |
| Default Network Ports | RPC: 8332 / P2P: 8333 | TCP daemon bindings for API and peer gossip |

---

<h2>2. ARCHITECTURAL COMPARISON: BITCOIN CORE VS RFSC</h2>

| Design Dimension | Bitcoin Core (C++) | RFSC Core (Native RFM) | Technical Advantage |
| :--- | :--- | :--- | :--- |
| Consensus Workload | Double SHA-256 loop | Dual-Tier PoUW AI Engine | Deterministic matrix operations and activations |
| Hardware Resistance | ASIC Dominated | Commodity GPU / CPU Friendly | Tensor GEMM demands high memory bandwidth |
| Ledger Storage | Google LevelDB (LSM) | NenoDB WAL Engine | 5.3x faster than LevelDB; 64KB Ring-Buffered WAL |
| Ledger Read Path | Disk-bound cache queries | Zero-I/O RAM Pinning | Hot UTXO set retained in RAM via native pin() |
| Transaction Privacy | Public plaintext values | Pedersen Commitments | Blinding factors conceal value while proving balance |
| P2P Socket Framing | Boost / libevent C++ | Linux Socket Syscalls | Direct kernel-level non-blocking socket wrappers |
| Project Toolchain | Autotools / CMake | RSC Project Orchestrator | Native AOT compilation via Clang-23 / LLVM backend |

---

<h2>3. REPOSITORY MODULE STRUCTURE</h2>

| Subsystem Component | Source Path | Lines | Primary Technical Scope |
| :--- | :--- | :---: | :--- |
| Consensus Kernel | src/core.rfm | 589 | Header primitives, UTXO validation, halving math, NenoDB WAL commits |
| Command-Line CLI | src/rfsc_cli.rfm | 503 | Wallet generation, node daemon, standalone mining, worker routines |
| JSON-RPC 2.0 Server | src/rpc_server.rfm | 206 | HTTP API daemon for wallet integrations, miners, and exchanges |
| Mempool Engine | src/mempool.rfm | 164 | Unconfirmed transaction pool, double-spend checks, fee prioritization |
| Peer-to-Peer Protocol | src/p2p.rfm | 134 | Node routing, gossip broadcast, peer registry, handshake logic |
| Transaction Model | src/transaction.rfm | 117 | CTxIn, CTxOut, OutPoint structs, double-hash TXID calculation |
| UTXO Ledger Cache | src/utxo_set.rfm | 93 | Fast in-memory ledger state with RAM pinning (zero disk read latency) |
| Socket Networking | src/net/p2p_socket.rfm | 89 | Linux socket syscall abstractions (listen, accept, read, write) |
| Core Type Definitions | src/types.rfm | 81 | Consensus constants, error structures, result enumerations |
| Network Wire Framing | src/wire.rfm | 69 | Binary wire frame serialization, header magic, payload validation |
| Zero-Knowledge Engine | src/zk_pedersen.rfm | 66 | Pedersen commitments and blinding factor generators |
| Cryptographic Engine | src/crypto.rfm | 63 | secp256k1 key derivation, Blake3 and SHA-256d routines |
| Node Configuration | src/config.rfm | 57 | Parameter parser for rfsc.conf configuration files |
| AI PoUW Tensor Engine | src/consensus/pouw_dual.rfm | 53 | Tensor GEMM matrix operations, GELU activation, Multi-Head Attention |
| Daemon Loop Service | src/net/rpc_daemon.rfm | 25 | Socket daemon loop for incoming HTTP JSON-RPC connections |

---

<h2>4. DUAL-TIER PROOF-OF-USEFUL-WORK (PoUW) CONSENSUS</h2>

| Processing Tier | Execution Schedule | Algorithmic Computation | Consensus Verification Target |
| :--- | :--- | :--- | :--- |
| Tier 1: Micro-Batch Nonce | Per Block Candidate | MatMul (W x X) + GELU Activation | Matrix trace binds to block header ai_loss_checksum |
| Tier 2: Macro-Epoch State | Every 2,016 Blocks | Multi-Head Attention (2 Heads) + RMSNorm | Aggregates global neural weights into epoch state root |

---

<h2>5. P2P WIRE PROTOCOL FRAME LAYOUT</h2>

| Field Name | Offset | Length | Data Type | Encoding / Value |
| :--- | :---: | :---: | :--- | :--- |
| Magic Bytes | 0x00 | 4 Bytes | uint32 (Big-Endian) | 0x52465343 (ASCII RFSC) |
| Message Type | 0x04 | 2 Bytes | uint16 (Big-Endian) | 0x01 Handshake, 0x02 Ping, 0x03 Block, 0x04 Tx |
| Payload Length | 0x06 | 4 Bytes | uint32 (Big-Endian) | Total byte count of payload data |
| Payload Checksum | 0x0A | 32 Bytes | byte[32] | Blake3 / SHA-256 integrity hash of payload |
| Message Payload | 0x2A | Variable | byte[] | Raw binary message payload |

---

<h2>6. CLUSTER MINING AND TREASURY VAULT ARCHITECTURE</h2>

| System Component | Network Role | Operational Specification |
| :--- | :--- | :--- |
| Central Coordinator | Master Node (Port 8332) | Issues candidate block templates with partitioned nonce spaces |
| Mining Workers | Distributed Machines | Compute distinct nonce intervals to eliminate duplicate effort |
| Worker Database | data/user.nenodb | Tracks accumulated Roc balances under unified parent account |
| Master Treasury Vault | On-Chain Reserve | Retains unreleased mining rewards until threshold verification |
| Payout Threshold | 100,000,000 Roc (1 RFSC) | Atomic WAL payout deduction triggered only upon reaching threshold |

---

<h2>7. VERIFICATION AND TEST SUITES</h2>

| Test Suite Module | Target File | Code Volume | Verification Result |
| :--- | :--- | :---: | :---: |
| Cryptographic Primitives | tests/test_crypto.rfm | 61 Lines | PASS |
| Transaction Processing | tests/test_transaction.rfm | 119 Lines | PASS |
| Mempool Ingestion | tests/test_mempool.rfm | 109 Lines | PASS |
| Peer-to-Peer Protocol | tests/test_p2p.rfm | 66 Lines | PASS |
| JSON-RPC API Interface | tests/test_rpc.rfm | 83 Lines | PASS |
| Zero-Knowledge Proofs | tests/test_zk.rfm | 73 Lines | PASS |
| Full Blockchain Lifecycle | tests/test_rfsc_full.rfm | 167 Lines | PASS |
| PoUW AI Tensor Engine | tests/test_pouw_ai.rfm | 98 Lines | PASS |

---

<h2>8. COMMAND-LINE INTERFACE (CLI) MANUAL</h2>

| Command Form | Parameters | Operational Output |
| :--- | :--- | :--- |
| rfsc-cli wallet new | None | Generates secp256k1 private key, pubkey, and rfsc1 bech32 address |
| rfsc-cli wallet balance | [address] [host] [port] | Reads UTXO balance from local NenoDB or coordinator endpoint |
| rfsc-cli node start | [rpc_port] [p2p_port] [db_path] | Spawns node coordinator (Default: RPC 8332, P2P 8333, NenoDB WAL) |
| rfsc-cli node info | [host] [port] | Queries blockchain height, tip hash, and remaining monetary supply |
| rfsc-cli miner start | [address] [blocks] [mode] | Executes standalone miner using native PoUW AI tensor routines |
| rfsc-cli miner worker | [host] [port] [addr] [n] [mode] | Connects worker instance to coordinator to fetch and solve jobs |
| rfsc-cli p2p listen | [port] | Binds TCP socket for incoming binary wire frame exchanges |
| rfsc-cli p2p send | [host] [port] [payload] | Transmits raw binary wire frame to remote peer |
| rfsc-cli p2p ping | [peer_id] | Emits heartbeat wire packet to evaluate network latency |

---

<h2>9. JSON-RPC 2.0 PROTOCOL INTERFACE</h2>

| RPC Method | Request Type | Parameters | Return Structure |
| :--- | :---: | :--- | :--- |
| getblockcount | POST | None | Integer representing current blockchain height |
| getbestblockhash | POST | None | 32-byte hexadecimal hash of the latest block |
| getblocktemplate | POST | [miner_address] | Block template with previous hash, height, and target |
| submitblock | POST | [block_hex] | Validation string (ACCEPTED or REJECTED) |
| sendrawtransaction | POST | [tx_hex] | 32-byte TXID string on successful mempool admission |
| getbalance | POST | [address] | Aggregate balance returned in Roc and RFSC formats |

---

<h2>10. CONFIGURATION PARAMETERS (rfsc.conf)</h2>

| Parameter Key | Default Value | Technical Purpose |
| :--- | :--- | :--- |
| rpc_port | 8332 | Listening TCP port for JSON-RPC 2.0 interface |
| p2p_port | 8333 | Listening TCP port for binary wire protocol peer connections |
| db_path | data/blockchain.nenodb | File system path for NenoDB WAL persistent storage |
| user_db_path | data/user.nenodb | File system path for cluster mining worker credentials and balances |
| coordinator_host | 127.0.0.1 | Target address of master node for cluster mining workers |
| mining_mode | deep | PoUW AI tensor compute precision (micro or deep) |
| min_payout_threshold | 100000000 | Minimum accumulated Roc (1 RFSC) required for treasury distribution |

---

<h2>11. TOOLCHAIN AND COMPILATION REFERENCE</h2>

| Action | Execution Command | Output Artifact |
| :--- | :--- | :--- |
| Project Build (Release) | rsc build --release | target/release/rfsc-cli |
| Project Build (Debug) | rsc build | target/debug/rfsc-cli |
| Direct Execution | rsc run --release -- [args] | Executes release binary with runtime arguments |
| Automated Test Runner | rsc test | Executes all integrated test suites |

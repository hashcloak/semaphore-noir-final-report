# Benchmarks for Semaphore Noir

A [benchmarking library](https://github.com/hashcloak/semaphore-noir-benchmarks) for Semaphore is provided that will output the benchmarks of the SDK in both node and browser environments. All of those numbers are averages over 10 runs. Additionally, we have benchmarked the gate count directly on the Noir circuit with bb gates.

## Machine Details

- Computer: MacBook Air
- Chip: Apple M2 (8-core, 3.49 GHz)
- Memory (RAM): 16 GB
- Operating System: macOS Sequoia version 15.3.1

## Version Details
- nargo 1.0.0-beta.3
- bb 0.82.2
- @aztec/bb.js 0.82.2
- @noir-lang/noir_js 1.0.0-beta.3
- @noir-lang/noir_wasm 1.0.0-beta.3

## Node Benchmarks
Below we benchmark generating and verifying Semaphore proofs with  N-member groups, using circuits with `MAX_DEPTH` set to K (see [Noir circuit](./semaphore_noir.md#noir-circuit)) in Node environment. Find and rerun the different benchmarking scripts [here](https://github.com/hashcloak/semaphore-noir-benchmarks/tree/main/node/src).

The benchmarks are split based on whether the proving backend is pre-initialized or not. In all cases we let the prover backend run on the highest possible number of threads (with `os.cpus().length`) for best performance. There are also benchmarks for the time it costs to initialize the backend. 

In practice an application could start loading the proving backend right away; this does not have to happen at the moment of actually creating the proof. Furthermore, this initialization only has to happen once and then proof generation (or verification) can continuously use that backend. 

### Benchmarks with backend pre-initialized

**Generate proofs** with proving backend already initialized. (see [Semaphore SDK](./semaphore_noir.md#semaphore-sdk))

| Function                                         | Avg Time (ms) |
| ------------------------------------------------ | ------------- |
| Generate Proof 1 Member \[Max tree depth 1]      | 296.87        |
| Generate Proof 100 Members \[Max tree depth 7]   | 412.60        |
| Generate Proof 500 Members \[Max tree depth 9]   | 454.38        |
| Generate Proof 1000 Members \[Max tree depth 10] | 483.66        |
| Generate Proof 2000 Members \[Max tree depth 11] | 538.07        |

**Verify proofs** with proving backend already initialized. (see [Semaphore SDK](./semaphore_noir.md#semaphore-sdk))

| Function                                       | Avg Time (ms) |
| ---------------------------------------------- | ------------- |
| Verify Proof 1 Member \[Max tree depth 1]      | 14.85         |
| Verify Proof 100 Members \[Max tree depth 7]   | 15.12         |
| Verify Proof 500 Members \[Max tree depth 9]   | 15.10         |
| Verify Proof 1000 Members \[Max tree depth 10] | 15.06         |
| Verify Proof 2000 Members \[Max tree depth 11] | 15.47         |

### Benchmarks without pre-initializing the proving backend

**Generate proofs** (includes backend initialization).

| Function                                                              | Avg Time (ms) |
| --------------------------------------------------------------------- | ------------- |
| Generate Proof 1 Member + Initialize backend \[Max tree depth 1]      | 547.23        |
| Generate Proof 100 Members + Initialize backend \[Max tree depth 7]   | 705.19        |
| Generate Proof 500 Members + Initialize backend \[Max tree depth 9]   | 760.26        |
| Generate Proof 1000 Members + Initialize backend \[Max tree depth 10] | 822.15        |
| Generate Proof 2000 Members + Initialize backend \[Max tree depth 11] | 885.36        |


**Verify proofs** (includes backend initialization).

| Function                                                            | Avg Time (ms) |
| ------------------------------------------------------------------- | ------------- |
| Verify Proof 1 Member + Initialize backend \[Max tree depth 1]      | 194.27        |
| Verify Proof 100 Members + Initialize backend \[Max tree depth 7]   | 224.15        |
| Verify Proof 500 Members + Initialize backend \[Max tree depth 9]   | 233.42        |
| Verify Proof 1000 Members + Initialize backend \[Max tree depth 10] | 237.17        |
| Verify Proof 2000 Members + Initialize backend \[Max tree depth 11] | 247.10        |


### UltraHonkBackend Initialization
The time to initialize proving backends with different circuits.

| Function                         | Avg Time (ms) |
| -------------------------------- | ------------- |
| Initialize Backend Tree Depth 1  | 185.58        |
| Initialize Backend Tree Depth 10 | 222.44        |
| Initialize Backend Tree Depth 20 | 276.05        |
| Initialize Backend Tree Depth 32 | 365.78        |

## Node benchmarks of the Original Semaphore V4

To compare with the original Semaphore implementation using Circom, we include the benchmarks below, all run on the same machine. Both proof generation and verification are around 1.4-1.7x slower for Semaphore Noir, if we assume that the proving backend for Noir will be pre-initialized. Proof generation with Noir ranges from 296-538ms, while the Circom implementation benches between 175-387ms for the different tree depths. Proof verification is 14-15ms versus 8ms. 

### Proof generation

| Function                    | Avg Time (ms) |
| --------------------------- | ------------- |
| Generate Proof 1 Member     | 175.45        |
| Generate Proof 100 Members  | 284.79        |
| Generate Proof 500 Members  | 313.28        |
| Generate Proof 1000 Members | 319.75        |
| Generate Proof 2000 Members | 387.23        |

### Proof verification

| Function                  | Avg Time (ms) |
| ------------------------- | ------------- |
| Verify Proof 1 Member     | 8.58          |
| Verify Proof 100 Members  | 8.59          |
| Verify Proof 500 Members  | 8.62          |
| Verify Proof 1000 Members | 8.64          |
| Verify Proof 2000 Members | 8.61          |


## Browser Benchmarks

The browser benchmarks are ran with pre-initialized proving backend. The script can be checked [here](https://github.com/hashcloak/semaphore-noir-benchmarks/blob/main/browser/src/utils/generate-benchmarks.ts). 

Proof generation with Noir in browser environment takes about 1-2secs, while Semaphore in Circom only takes about 200~300ms. This is likely caused by the memory limit of the browser, since UltraHonk uses more memory than Groth16 which is used in Circom. It is a future research to optimize Semaphore Noir as well as UltraHonk in the browser environment.

### Proof generation

| Function                                         | Avg Time (ms) |
| ------------------------------------------------ | ------------- |
| Generate Proof 1 Member \[Max tree depth 1]      | 797.18        |
| Generate Proof 100 Members \[Max tree depth 7]   | 1315.42       |
| Generate Proof 500 Members \[Max tree depth 9]   | 1457.08       |
| Generate Proof 1000 Members \[Max tree depth 10] | 1528.59       |
| Generate Proof 2000 Members \[Max tree depth 11] | 1815.77       |

### Comparison with original Semaphore

| Function                                         | Avg Time (ms) |
| ------------------------------------- | ------------- |
| Generate Proof 1 Member               | 167.91        |
| Generate Proof 100 Members            | 276.99        |
| Generate Proof 500 Members            | 305.99        |
| Generate Proof 1000 Members           | 317.27        |
| Generate Proof 2000 Members           | 397.70        |

## Gas estimates
Estimations of gas usage in the SemaphoreNoir contract.

| Function                                                        |Gas Usage|
|-----------------------------------------------------------------|---------|
| SemaphoreNoir.verifyProof 1 Member [Max tree depth 1]           |1856690  |
| SemaphoreNoir.verifyProof 100 Members [Max tree depth 7]        |1887944  |
| SemaphoreNoir.verifyProof 500 Members [Max tree depth 9]        |1887800  |
| SemaphoreNoir.verifyProof 1000 Members [Max tree depth 10]      |1887896  |
| SemaphoreNoir.verifyProof 2000 Members [Max tree depth 11]      |1919115  |


| Function                                                        |Gas Usage|
|-----------------------------------------------------------------|---------|
| SemaphoreNoir.validateProof 1 Member [Max tree depth 1]         |1996899  |
| SemaphoreNoir.validateProof 100 Members [Max tree depth 7]      |2027997  |
| SemaphoreNoir.validateProof 500 Members [Max tree depth 9]      |2028057  |
| SemaphoreNoir.validateProof 1000 Members [Max tree depth 10]    |2028093  |
| SemaphoreNoir.validateProof 2000 Members [Max tree depth 11]    |2059288  |

### Comparison with `Semaphore.sol`

In the `Semaphore.sol` contract for V4 of Semaphore, calling `validateProof` for 10 members has an estimated gas cost of 285622, and this number is 285598 for 30 members ([benchmark reference](https://docs.semaphore.pse.dev/benchmarks#contracts)). Although the above benchmarks don't show exactly those group sizes, we can estimate that validation in `SemaphoreNoir.sol` is approximately 7x more expensive. In the next section we'll discuss how batching for Noir can be applied to significantly lower the gas cost if the Semaphore proofs get validated in batches rather than individually. 

## Circuit Gate Counts
Gate counts of the Semaphore Noir circuit of different `MAX_DEPTH`s.

| MAX_DEPTH | acir_opcodes | circuit_size |
|-----------|--------------|--------------|
|         1 |         2822 |         7756 |
|         2 |         3149 |         8696 |
|         3 |         3476 |         9636 |
|         4 |         3803 |        10576 |
|         5 |         4130 |        11516 |
|         6 |         4457 |        12456 |
|         7 |         4784 |        13397 |
|         8 |         5111 |        14336 |
|         9 |         5438 |        15276 |
|        10 |         5765 |        16217 |
|        11 |         6092 |        17157 |
|        12 |         6419 |        18096 |
|        13 |         6746 |        19037 |
|        14 |         7073 |        19977 |
|        15 |         7400 |        20917 |
|        16 |         7727 |        21857 |
|        17 |         8054 |        22797 |
|        18 |         8381 |        23737 |
|        19 |         8708 |        24678 |
|        20 |         9035 |        25617 |
|        21 |         9362 |        26557 |
|        22 |         9689 |        27498 |
|        23 |        10016 |        28438 |
|        24 |        10343 |        29377 |
|        25 |        10670 |        30318 |
|        26 |        10997 |        31258 |
|        27 |        11324 |        32198 |
|        28 |        11651 |        33138 |
|        29 |        11978 |        34078 |
|        30 |        12305 |        35018 |
|        31 |        12632 |        35959 |
|        32 |        12959 |        36898 |

## ZK artifact sizes

In the SDK to generate a Semaphore or Semaphore Noir proof compiled versions of the respective circuits (Circom or Noir) are used. These are called "ZK artifacts" in the Semaphore protocol. In the case of Circom, this consists of a wasm and a json file and in the case of Noir only a json file. 

For Semaphore Noir these artifacts are used to [instantiate](https://github.com/hashcloak/semaphore-noir/blob/main/packages/proof/src/semaphore-noir-backend.ts#L26) the proving backend, which is then passed on to the proving functionality. [This](https://github.com/hashcloak/semaphore-noir/blob/main/packages/proof/src/semaphore-noir-backend.ts#L36) is where the Noir artifacts are retrieved. For the original Semaphore (Circom) implementation, downloading the artifacts is done [here](https://github.com/semaphore-protocol/semaphore/blob/main/packages/proof/src/generate-proof.ts#L87). 

Comparing these ZK artifacts in size, it’s clear that Noir is more lightweight than Circom. Circom requires both a .json file (around ~3.66 KB) and a .wasm file (ranging from 1.71 MB to 1.77 MB for tree depths 1–32). Noir requires only a .json file, with size increasing gradually from 259 KB to 1.52 MB depending on the tree depth. 

The table below compares selected depths and shows that Noir ZK artifacts are consistently smaller.


| Tree Depth | Noir `.json` | Circom `.json` | Circom `.wasm` |
| ---------- | ------------ | -------------- | -------------- |
| 1          | 259 KB       | 2.66 KB        | 1.71 MB        |
| 2          | 301 KB       | 3.66 KB        | 1.71 MB        |
| 3          | 342 KB       | 3.66 KB        | 1.71 MB        |
| 4          | 384 KB       | 3.65 KB        | 1.71 MB        |
| 5          | 426 KB       | 3.66 KB        | 1.71 MB        |
| 10         | 635 KB       | 3.66 KB        | 1.72 MB        |
| 15         | 844 KB       | 3.66 KB        | 1.75 MB        |
| 20         | 1.03 MB      | 3.65 KB        | 1.75 MB        |
| 30         | 1.44 MB      | 3.66 KB        | 1.76 MB        |
| 32         | 1.52 MB      | 3.65 KB        | 1.77 MB        |

The Noir ZK artifacts can be found [here](https://github.com/hashcloak/snark-artifacts/tree/main/packages/semaphore-noir) and the original artifacts can be found [here](https://github.com/privacy-scaling-explorations/snark-artifacts/tree/main/packages/semaphore). 
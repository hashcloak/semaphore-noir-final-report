# Benchmarks Batching Noir proofs

**[Draft report] Note:** these benchmarks are for the batching setting with the trusted backend. We will update them when the improved functionality is ready. 

In the [benchmarking library](https://github.com/hashcloak/semaphore-noir-benchmarks/blob/main/node/src/batching.ts) you can find the functionality to rerun these benchmarks for batching. 

The average values below are over 3 runs and always generate the final proof with the `keccak` flag, so it is ready for on-chain verification. The most important benchmarks here are batch proof generation and the gas cost estimates for on-chain verification, because the main aim for batching is to lower the total gas cost. 

## Batch Proof Generation

Generation of a batch proof of `N` Semaphore proofs using the function [`batchSemaphoreNoirProofs`](https://github.com/hashcloak/semaphore-noir/blob/noir-support-part2/packages/noir-proof-batch/src/batch.ts#L63C31-L63C55) with `keccak` enabled for on-chain use.

| Function              | Avg time (ms) | Avg time (min) |
| --------------------- | ------------- | -------------- |
| Generate batch of 10  | 96004         | 1.60           |
| Generate batch of 20  | 223294        | 3.72           |
| Generate batch of 30  | 355233        | 5.92           |
| Generate batch of 100 | 1233135       | 20.55          |

## Semaphore proof generation for batching

Since batching uses recursion, we need to generate the Semaphore proof in a different way. For this we have added [`generateNoirProofForBatching`](https://github.com/hashcloak/semaphore-noir/blob/noir-support-part2/packages/noir-proof-batch/src/generate-proof-noir.ts#L37) which generates a proof using the bb CLI, see the benchmarks below (benchmark script [reference](https://github.com/hashcloak/semaphore-noir-benchmarks/blob/main/node/src/generate-proof-for-batching.ts)). This form of Semaphore proof generation is slightly faster than the benchmarks we shared above, because bb CLI is faster than `bb.js`. 

| Function                                                        | Avg Time (ms) |
| --------------------------------------------------------------- | ------------- |
| Generate Proof (for batching) 1 Member \[Max tree depth 1]      | 238.81        |
| Generate Proof (for batching) 100 Members \[Max tree depth 7]   | 307.53        |
| Generate Proof (for batching) 500 Members \[Max tree depth 9]   | 336.82        |
| Generate Proof (for batching) 1000 Members \[Max tree depth 10] | 350.36        |
| Generate Proof (for batching) 2000 Members \[Max tree depth 11] | 389.08        |


## Gas estimates on-chain verification on a Batch proof

**[Draft report] Note:** when the improved version is done, the contract will also have a validateProof function that checks the nullifiers & merkle root. The current version doesn't have that yet. We'll add those benchmarks here when it's done. 

When generating a smart contract verifier for a Noir circuit using CLI, this contract has a verifier function:
```rust=
interface IVerifier {
    function verify(bytes calldata _proof, bytes32[] calldata _publicInputs) external view returns (bool);
}
```

To fully verify a Semaphore proof, as well as a Batch proof there are additional checks to be done. In the current version of batching we haven't added those checks yet, because we aim to eliminate the trusted backend that was needed. 

The earlier gas estimates given are all with respect to the SemaphoreNoir smart contract, which include the extra checks. The estimated gas cost for just the `verify` function there is ~1747700.

For a Batch proof the estimated gas cost is ~1985000. 
This means that verifying a Batch proof is only ~1.13x more expensive than verifying a Semaphore proof. Note that this part of the verification cost for a batch proof won't depend on how many Semaphore proofs have been batched together. (The additional checks on nullifiers and merkle proofs *will* depend on the number of input proofs)

## Batch Proof Verification (SDK)

On the application side, it is possible we'd like to verify the batch proof locally before verifying on-chain. This is slightly more costly than verifying a "normal" Semaphore proof with the SDK. Benchmarks for [`verifyBatchProof`](https://github.com/hashcloak/semaphore-noir/blob/noir-support-part2/packages/noir-proof-batch/src/batch-verify.ts#L12):

| Function           | Avg time (ms) |
| ------------------ | :-----------: |
| Verify batch of 10 |      46       |
| Verify batch of 20 |      38       |
| Verify batch of 30 |      32       |
| Verify batch of 100|      40       |
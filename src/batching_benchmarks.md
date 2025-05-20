# Benchmarks Batching Noir proofs

In the [benchmarking library](https://github.com/hashcloak/semaphore-noir-benchmarks/blob/main/node/src/batching.ts) you can find the functionality to rerun these benchmarks for batching. 

The average values below are over 3 runs and always generate the final proof with the `keccak` flag, so it is ready for on-chain verification. The most important benchmarks here are batch proof generation and the gas cost estimates for on-chain verification, because the main aim for batching is to lower the total gas cost. 

## Batch Proof Generation

Generation of a batch proof of `N` Semaphore proofs using the function [`batchSemaphoreNoirProofs`](https://github.com/hashcloak/semaphore-noir/blob/noir-support-part2/packages/noir-proof-batch/src/batch.ts#L63C31-L63C55) with `keccak` enabled for on-chain use.

| Function              | Avg time (ms) | Avg time (min) |
| --------------------- | ------------- | -------------- |
| Generate batch of 10  | 97064         | 1.62           |
| Generate batch of 20  | 214852        | 3.58           |
| Generate batch of 30  | 349871        | 5.83           |
| Generate batch of 100 | 1187020       | 19.78          |

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

In the `SemaphoreNoir` smart contract, a batch proof can be validated with `validateBatchedProof`. This will verify the batch proof, check that there are no duplicated nullifiers and that all the used merkle roots are correct. 

For a single Semaphore Noir proof for a 100 member group, calling `validateProof` costs an estimate of 2027997 in gas. If we validate a batch of Semaphore Noir proofs, the gas usage is as follows for groups of 100 members:

| Function                                      | Gas Usage | Comparison   |
| --------------------------------------------- | --------- | :------------: |
| SemaphoreNoir.validateBatchedProof 10 proofs  | 2689415   | 1.33x        |
| SemaphoreNoir.validateBatchedProof 20 proofs  | 3095601   | 1.53x        |
| SemaphoreNoir.validateBatchedProof 30 proofs  | 3572773   | 1.76x        |
| SemaphoreNoir.validateBatchedProof 50 proofs  | 4745292   | 2.34x        |
| SemaphoreNoir.validateBatchedProof 100 proofs | 8933459   | 4.41x        |

As the table shows, batching significantly reduces the per-proof gas cost. For example, 100 proofs can be verified on-chain for only 4.4 times the gas cost of verifying a single proof. As shown earlier, generating such a batch could take around 20 minutes, showing that there's a clear tradeoff between computation effort and time off-chain and gas saved on-chain.

## Batch Proof Verification (SDK)

On the application side, it is possible we'd like to verify the batch proof locally before verifying on-chain. This is slightly more costly than verifying a "normal" Semaphore proof with the SDK. Benchmarks for [`verifyBatchProof`](https://github.com/hashcloak/semaphore-noir/blob/noir-support-part2/packages/noir-proof-batch/src/batch-verify.ts#L12):

| Function           | Avg time (ms) |
| ------------------ | :-----------: |
| Verify batch of 10 |      46       |
| Verify batch of 20 |      32       |
| Verify batch of 30 |      31       |
| Verify batch of 100|      37       |
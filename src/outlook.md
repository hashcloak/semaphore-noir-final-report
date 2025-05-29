# Outlook

Wrapping up this project, we'd like to share some insights and ideas for future research or other implementation work as well as discuss a bit about what this current project unlocks. 

## Unlocked applications

The Semaphore Noir application with batching can make a big difference for on-chain voting gas costs in comparison to using the existing Semaphore protocol. Any application that aims to use Semaphore with on-chain verification could prefer the tradeoff that batching offers by spending some extra time on batching the proofs, but having a much lower verification cost. The cost to verify a Noir batch proof will outweigh the cost of verifying separate Circom Semaphore proofs after batching more than ~10 proofs. This means it will mainly be attractive to use for applications that work with a large number of proofs. 

## Semaphore Proof of non-membership with Noir

In the research [notes](https://pse-team.notion.site/Semaphore-V4-Research-683cd07c4c1a4271bc555ba296ebefa3) for Semaphore version 4, there is a feature idea for proof of non-membership. This would allow people who are *not* part of a certain group to send signals. However, this is not easy to implement with the current structure for groups. We think that with Noir, and specifically recursion in Noir, proof of non-membership could be implemented for Semaphore V4 for small groups.

The members of a group in Semaphore are the leaves of a binary tree (more specifically, a LeanIMT). Each leaf is a hash of the public key of the member. To show you are not part of the group you can show that the hash of your public key is not equal to any of the leaves. If we wrap this in Noir proofs for each leaf and batch them together similarly to how we did it in this project, we think this could be practical for small groups < 100 members. 

How could we implement this? For each leaf `L_i` of the leanIMT, generate a proof that `L_i` is part of the group and that my leaf `x` is not equal to it. We can call these our NotLeafProofs. 

Then, we can create a batching circuit that takes 2 NotLeafProofs, verifies both proofs and outputs their respective leaves `L_i`. Finally we have another batching circuit that verifies previous proofs and propagates the hash of the leaves `L_i`. If we follow the same strategy as the LeanIMT, promoting a single node to the next level, and thus promoting the hash of the leaves to the next level, the resulting hash should be equal to the merkle root of the group. 

The end result would be a final Batch proof which can be verified with the merkle root. This merkle root should be equal to the most recent merkle root known in the smart contract (otherwise the leaf could be a newer member). The cool thing is that of course we can also batch these proofs together and ultimately proving multiple non-membership proofs on-chain at once. 

## Feature suggestions for Noir & bb.js

The first feature request was already mentioned in our [progress report](https://hashcloak.github.io/semaphore-noir-progress-report/): it would be great if a Noir circuit can be [parameterized](https://hashcloak.github.io/semaphore-noir-progress-report/discoveries.html#parameterizing-the-circuit-in-noir-vs-circom). In the case of Semaphore in Noir there are practically speaking 32 variations of the same circuit, with the difference being a constant which indicated the maximum depth of the merkle tree: `pub global MAX_DEPTH: u32 = 10;`. Currently, we overwrite this value in a script in order to obtain the circuit we want. It would be great if circuit parameterization would be added to a feature version of Noir! This feature request was already made in GitHub, follow status [here](https://github.com/noir-lang/noir/issues/3858). 

In addition, in Semaphore Noir we are using an adapted version of the `UltraHonkBackend`, see code reference [here](https://github.com/hashcloak/semaphore-noir/blob/noir-support-part2/packages/proof/src/ultrahonk.ts#L32). To increase performance we allow users to reuse verification keys, which required a small adjustment to this class. It is imaginable more developers would like to make use of this option, so maybe this could be added in a future `bb.js` version. We added this issue in GitHub [here](https://github.com/noir-lang/noir/issues/8717). 

## Parallelizing batching of Semaphore proofs

The current benchmarks for batching of Semaphore proofs does not take into account parallelization. Especially for the first layer of generating the initial Batch proofs, this could make for a significant performance improvement. In general, for batching large number of Semaphore proofs it would be interesting to see how much of an improvement parallelization could mean. 

## Add Semaphore Noir template to Remix IDE

The Semaphore Circom template was [added](https://medium.com/remix-ide/remix-release-v0-37-0-dbc750f7ab15) to Remix IDE a few years ago. More recently, also [Noir support](https://medium.com/remix-ide/remix-release-v0-59-0-306881e41984) was added to Remix! We'd like to leave this observation here in case it is possible and desirable to add the Semaphore Noir template to Remix. 

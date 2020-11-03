# Install the Frontier Block Import

In this section we'll install the `FrontierBlockImport` in the node. This is the first change we are
making outside of the runtime. In the Frontier repository, this crate is called "frontier-consensus".
Although the block import pipeline is most typically used by consensus-related tasks, we are not using
it in that way, and so I'll avoid the term consensus in this writeup.

For reference, you can see this work in commit [1c0931c](https://github.com/JoshOrndorff/substrate-node-template/commit/1c0931c59122d02b6ee8b3c55d35532f2c2174ce).

## Snippets

`node/Cargo.toml`
```toml
frontier-consensus = { git = 'https://github.com/PureStake/frontier.git', branch = 'substrate-v2' }
```

`node/src/service.rs`
```rust
use frontier_consensus::FrontierBlockImport;
```

Update the type of our block import pipeline in the return type of the `new_partial` function.
`node/src/service.rs`
```rust
sc_consensus_aura::AuraBlockImport<
			Block,
			FullClient,
			FrontierBlockImport<
				Block,
				sc_finality_grandpa::GrandpaBlockImport<FullBackend, Block, FullClient, FullSelectChain>,
				FullClient
			>,
			AuraPair
		>
```

Install the block import pipeline.
`node/src/service.rs`
```rust
// Here we inert a piece in the block import pipeline
// The old pipeline was Aura -> Grandpa -> Client
// The new pipeline is Aura -> Frontier -> Grandpa -> Client
let frontier_block_import = FrontierBlockImport::new(
	grandpa_block_import.clone(),
	client.clone(),
	true
);

// Be sure to update the existing aura block import to use the new piece.
let aura_block_import = sc_consensus_aura::AuraBlockImport::<_, _, _, AuraPair>::new(
	frontier_block_import.clone(), client.clone(),
);
```


## Helpful Resources

* Block import pipeline docs https://substrate.dev/docs/en/knowledgebase/advanced/block-import
* Frontier consensus code https://github.com/paritytech/frontier/tree/master/consensus

## Check Your Work

At this point the entire node should build with `cargo check`.

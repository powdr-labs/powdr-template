# powdrVM template

This template is a basic structure for a powdrVM host/guest project.

## Dependencies

- Rust/cargo

## Usage

This will run the host and generate ZK proofs.

```bash
cargo run -r
```

## AVX / Neon

You can enable AVX or Neon support by using the `simd` feature and running the host with extra flags:

```bash
RUSTFLAGS='-C target-cpu=native' cargo run --features simd -r
```

## Structure

- `src/main.rs`: the host code. This is where you create a powdr `Session`, prepare data to be shared with the guest, and run the prover.
- `guest`: this is the guest crate. It contains the code that will be run inside the powdrVM.
- `powdr-target`: this is where all generated artifacts reside. This includes the compiled guest code to powdr-asm, the compiled PIL constraints, setup artifacts such as proving and verifying keys, and the final ZK proofs.

## Workflow

Let's look at `src/main.rs` line by line:

Here we create some data we want to share with the guest:
```rust
let some_data = vec![1, 2, 3, 4, 5];
```

Create a new powdr session where we'll be running crate `guest` in powdrVM
and all artifacts will be stored in `powdr-target`:
```rust
let mut session = Session::new("./guest", "powdr-target")
```

Write `some_data` to channel 1 and the sum of `some_data` to channel 2.
The guest will read this data from the channels:
```rust
.write(1, &some_data).write(2, &some_data.iter().sum::<u32>());
```

Run the session without generating a proof. Useful for testing the guest code:
```rust
session.run();
```

Generate the ZK proof:
```rust
session.prove();
```

Before generating a proof, powdrVM has to create the proving and verifying keys (setup) for the given guest program. When run for the first time, this can take a while. Subsequent runs will be faster as the setup only changes if the guest changes.

You can also run the host with INFO logs to have a deeper look at what's happening:
```bash
RUST_LOG=info cargo run -r
```

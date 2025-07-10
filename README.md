# Shamir Vault

`shamir-vault` is a Rust crate that provides an implementation of Shamir's Secret Sharing algorithm, enabling secure splitting and reconstruction of secrets. This crate allows you to divide a secret into multiple shares and reconstruct it with a minimum threshold of shares, ensuring data security and redundancy.

## Features

- Split secrets into `n` shares with a threshold `t` required for reconstruction.
- Robust error handling for input validation.
- Implementation using Galois Field arithmetic for security and efficiency.
- Easy-to-use API with comprehensive test coverage.
- **`no_std` compatible** - works in embedded and WASM environments
- **Explicit RNG control** - you provide the randomness source for better security

## Installation

Add `shamir-vault` to your `Cargo.toml` dependencies:

```toml
[dependencies]
shamir-vault = "1.2.0"
```

For `no_std` environments, both dependencies will automatically work without their default features.

## Usage

### 1. Splitting a Secret

You can split a secret into multiple shares, requiring a specified threshold for reconstruction. You must provide your own RNG instance.

```rust
use shamir_vault::{split, combine};

fn main() {
    let secret = b"My Super Secret Data";
    let mut rng = rand::thread_rng();
    let shares = split(secret, 5, 3, &mut rng).expect("Failed to split secret");
    
    println!("Generated Shares:");
    for (i, share) in shares.iter().enumerate() {
        println!("Share {}: {:?}", i + 1, share);
    }
}
```

**Parameters:**
- `secret`: A byte array representing the secret.
- `shares`: The total number of shares to generate.
- `threshold`: The minimum number of shares required to reconstruct the secret.
- `rng`: A mutable reference to any RNG implementing `RngCore`.

**Errors:**
- `InvalidShareCount`: If shares are not between 2 and 255.
- `InvalidThreshold`: If the threshold is not between 2 and 255.
- `SharesLessThanThreshold`: If the number of shares is less than the threshold.
- `EmptySecret`: If the secret is empty.

### 2. Reconstructing a Secret

To recover the original secret, provide at least the threshold number of shares.

```rust
use shamir_vault::{split, combine};

fn main() {
    let secret = b"My Super Secret Data";
    let mut rng = rand::thread_rng();
    let shares = split(secret, 5, 3, &mut rng).expect("Failed to split secret");
    
    let recovered_secret = combine(&shares[0..3]).expect("Failed to reconstruct secret");
    
    assert_eq!(secret, recovered_secret.as_slice());
    println!("Recovered Secret: {:?}", String::from_utf8_lossy(&recovered_secret));
}
```

**Parameters:**
- `shares`: A slice of shares used for reconstruction.

**Errors:**
- `InconsistentShareLength`: If shares have varying lengths.
- `DuplicateShares`: If there are duplicate shares.
- `ShareCountMismatch`: If the provided shares count does not match the required count.

### 3. Handling Errors

This crate provides robust error handling with the `ShamirError` enum.

```rust
use shamir_vault::{split, ShamirError};

fn main() {
    let mut rng = rand::thread_rng();
    match split(b"", 5, 3, &mut rng) {
        Ok(_) => println!("Secret successfully split"),
        Err(ShamirError::EmptySecret) => println!("Secret cannot be empty"),
        Err(e) => println!("Error: {}", e),
    }
}
```

### 4. Using in `no_std` Environments

For embedded systems or WASM, you can use any RNG that implements `RngCore`:

```rust
#![no_std]
extern crate alloc;

use shamir_vault::{split, combine};
use alloc::vec::Vec;
use rand_core::RngCore;

// Example with a custom RNG or hardware RNG
fn split_secret_embedded(mut rng: impl RngCore) {
    let secret = b"My Super Secret Data";
    let shares = split(secret, 5, 3, &mut rng).expect("Failed to split secret");
    // ... use shares
}
```

## API Documentation

### Functions

#### `split(secret: &[u8], shares: usize, threshold: usize, rng: &mut (dyn RngCore + '_)) -> Result<Vec<Vec<u8>>, ShamirError>`

Splits the given secret into a specified number of shares with a threshold for reconstruction.

#### `combine(shares: &[Vec<u8>]) -> Result<Vec<u8>, ShamirError>`

Combines the provided shares to reconstruct the original secret.

### Errors

#### `ShamirError`

- `InvalidShareCount`
- `InvalidThreshold`
- `SharesLessThanThreshold`
- `EmptySecret`
- `DuplicateShares`
- `InconsistentShareLength`
- `ShareCountMismatch`

## Security Considerations

- Ensure that secret shares are distributed securely to prevent unauthorized reconstruction.
- Use a sufficiently high threshold to prevent loss due to missing shares.
- Keep the number of generated shares within a reasonable limit (max 255).
- **Use a cryptographically secure RNG** - `rand::thread_rng()` is recommended for most applications.
- **Control your randomness source** - the explicit RNG parameter gives you full control over entropy.

## Performance

The crate is optimized for performance using precomputed Galois Field tables for fast arithmetic operations. The `no_std` design makes it suitable for resource-constrained environments.

## Testing

Unit tests are included to ensure the correctness of the implementation.

Run tests with:

```sh
cargo test
```

## Breaking Changes in v1.2.0

- **RNG Parameter Required**: The `split` function now requires an explicit RNG parameter
- **`no_std` Compatible**: Now works in `no_std` environments
- **Dependency Changes**: `rand` and `thiserror` now use `default-features = false`

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.

## Contribution

Feel free to submit issues, suggestions, or pull requests on GitHub: [shamir-vault](https://github.com/simplysabir/shamir-vault)

## Author

Developed by **Sabir Khan**


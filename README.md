<div align="center">

# Barcus Network

<img height="70px" alt="Polkadot SDK Logo" src="https://github.com/paritytech/polkadot-sdk/raw/master/docs/images/Polkadot_Logo_Horizontal_Pink_White.png#gh-dark-mode-only"/>
<img height="70px" alt="Polkadot SDK Logo" src="https://github.com/paritytech/polkadot-sdk/raw/master/docs/images/Polkadot_Logo_Horizontal_Pink_Black.png#gh-light-mode-only"/>

> A custom [parachain](https://wiki.polkadot.network/docs/learn-parachains) built with Polkadot SDK by [BioKeyper](https://github.com/biokeyper).
>
> **Parachain ID:** `1000`

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Built with Polkadot SDK](https://img.shields.io/badge/Built%20with-Polkadot%20SDK-E6007A)](https://github.com/paritytech/polkadot-sdk)

</div>

## Table of Contents

- [About](#about)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Starting a Development Chain](#starting-a-development-chain)
  - [Omni Node](#omni-node-prerequisites)
  - [Zombienet setup with Omni Node](#zombienet-setup-with-omni-node)
  - [Parachain Template Node](#parachain-template-node)
  - [Connect with the Polkadot-JS Apps Front-End](#connect-with-the-polkadot-js-apps-front-end)
  - [Takeaways](#takeaways)
- [Runtime Development](#runtime-development)
- [Contributing](#contributing)
- [Resources](#resources)

## About

**Barcus Network** is a blockchain parachain built using the [Polkadot SDK](https://github.com/paritytech/polkadot-sdk). It leverages the security and interoperability of the Polkadot relay chain while providing custom functionality through specialized pallets.

### Features

- 🔐 **Custom Pallets** - Specialized runtime modules tailored for Barcus Network
- ⚡ **High Performance** - Built on Substrate framework for optimal efficiency
- 🌐 **Interoperable** - Connects to Polkadot ecosystem via Cumulus
- 🛡️ **Secure** - Inherits security from Polkadot relay chain

### Technology Stack

- **Framework:** [Cumulus](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/polkadot_sdk/cumulus/index.html) (Substrate-based parachain framework)
- **Language:** Rust
- **Consensus:** Relay chain finality via Polkadot validators
- **Runtime:** Custom pallets with standard Substrate pallets ([Balances](https://paritytech.github.io/polkadot-sdk/master/pallet_balances/index.html), [System](https://paritytech.github.io/polkadot-sdk/master/frame_system/index.html), etc.)

> 👉 Learn more about parachains [here](https://wiki.polkadot.network/docs/learn-parachains)

## Project Structure

A Polkadot SDK based project such as this one consists of:

- 🧮 the [Runtime](./runtime/README.md) - the core logic of the parachain.
- 🎨 the [Pallets](./pallets/README.md) - from which the runtime is constructed.
- 💿 a [Node](./node/README.md) - the binary application, not part of the project default-members list and not compiled unless
  building the project with `--workspace` flag, which builds all workspace members, and is an alternative to
  [Omni Node](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/omni_node/index.html).

## Getting Started

- 🦀 The template is using the Rust language.

- 👉 Check the
  [Rust installation instructions](https://www.rust-lang.org/tools/install) for your system.

- 🛠️ Depending on your operating system and Rust version, there might be additional
  packages required to compile this template - please take note of the Rust compiler output.

Clone the Barcus Network repository:

```sh
git clone https://github.com/biokeyper/barcus-network.git

cd barcus-network
```

## Starting a Development Chain

The parachain template relies on a hardcoded parachain id which is defined in the runtime code
and referenced throughout the contents of this file as `{{PARACHAIN_ID}}`. Please replace
any command or file referencing this placeholder with the value of the `PARACHAIN_ID` constant:

```rust,ignore
pub const PARACHAIN_ID: u32 = 1000;
```

### Omni Node Prerequisites

[Omni Node](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/omni_node/index.html) can
be used to run the parachain template's runtime. `polkadot-omni-node` binary crate usage is described at a high-level
[on crates.io](https://crates.io/crates/polkadot-omni-node).

#### Install `polkadot-omni-node`

Please see the installation section at [`crates.io/omni-node`](https://crates.io/crates/polkadot-omni-node).

#### Build `parachain-template-runtime`

```sh
cargo build --profile production
```

#### Install `staging-chain-spec-builder`

Please see the installation section at [`crates.io/staging-chain-spec-builder`](https://crates.io/crates/staging-chain-spec-builder).

#### Use `chain-spec-builder` to generate the `chain_spec.json` file

```sh
chain-spec-builder create --relay-chain "rococo-local" --para-id {{PARACHAIN_ID}} --runtime \
    target/release/wbuild/parachain-template-runtime/parachain_template_runtime.wasm named-preset development
```

**Note**: the `relay-chain` and `para-id` flags are mandatory information required by
Omni Node, and for parachain template case the value for `para-id` must be set to `{{PARACHAIN_ID}}`, since this
is also the value injected through [ParachainInfo](https://docs.rs/staging-parachain-info/0.17.0/staging_parachain_info/)
pallet into the `parachain-template-runtime`'s storage. The `relay-chain` value is set in accordance
with the relay chain ID where this instantiation of parachain-template will connect to.

#### Run Omni Node

Start Omni Node with the generated chain spec. We'll start it in development mode (without a relay chain config), producing
and finalizing blocks based on manual seal, configured below to seal a block with each second.

```bash
polkadot-omni-node --chain <path/to/chain_spec.json> --dev --dev-block-time 1000
```

However, such a setup is not close to what would run in production, and for that we need to setup a local
relay chain network that will help with the block finalization. In this guide we'll setup a local relay chain
as well. We'll not do it manually, by starting one node at a time, but we'll use [zombienet](https://paritytech.github.io/zombienet/intro.html).

Follow through the next section for more details on how to do it.

### Zombienet setup with Omni Node

Assuming we continue from the last step of the previous section, we have a chain spec and we need to setup a relay chain.
We can install `zombienet` as described [here](https://paritytech.github.io/zombienet/install.html#installation), and
`zombienet-omni-node.toml` contains the network specification we want to start.

#### Relay chain prerequisites

Download the `polkadot` (and the accompanying `polkadot-prepare-worker` and `polkadot-execute-worker`) binaries from
[Polkadot SDK releases](https://github.com/paritytech/polkadot-sdk/releases). Then expose them on `PATH` like so:

```sh
export PATH="$PATH:<path/to/binaries>"
```

#### Update `zombienet-omni-node.toml` with a valid chain spec path

To simplify the process of using the parachain-template with zombienet and Omni Node, we've added a pre-configured
development chain spec (dev_chain_spec.json) to the parachain template. The zombienet-omni-node.toml file of this
template points to it, but you can update it to an updated chain spec generated on your machine. To generate a
chain spec refer to [staging-chain-spec-builder](https://crates.io/crates/staging-chain-spec-builder)

Then make the changes in the network specification like so:

```toml
# ...
[[parachains]]
id = "<PARACHAIN_ID>"
chain_spec_path = "<TO BE UPDATED WITH A VALID PATH>"
# ...
```

#### Start the network

```sh
zombienet --provider native spawn zombienet-omni-node.toml
```

### Parachain Template Node

As mentioned in the `Template Structure` section, the `node` crate is optionally compiled and it is an alternative
to `Omni Node`. Similarly, it requires setting up a relay chain, and we'll use `zombienet` once more.

#### Install the `parachain-template-node`

```sh
cargo install --path node
```

#### Setup and start the network

For setup, please consider the instructions for `zombienet` installation [here](https://paritytech.github.io/zombienet/install.html#installation)
and [relay chain prerequisites](#relay-chain-prerequisites).

We're left just with starting the network:

```sh
zombienet --provider native spawn zombienet.toml
```

### Connect with the Polkadot-JS Apps Front-End

- 🌐 You can interact with your local node using the
  hosted version of the Polkadot/Substrate Portal:
  [relay chain](https://polkadot.js.org/apps/#/explorer?rpc=ws://localhost:9944)
  and [parachain](https://polkadot.js.org/apps/#/explorer?rpc=ws://localhost:9988).

- 🪐 A hosted version is also
  available on [IPFS](https://dotapps.io/).

- 🧑‍🔧 You can also find the source code and instructions for hosting your own instance in the
  [`polkadot-js/apps`](https://github.com/polkadot-js/apps) repository.

### Takeaways

Development parachains:

- 🔗 Connect to relay chains, and we showcased how to connect to a local one.
- 🧹 Do not persist the state.
- 💰 Are preconfigured with a genesis state that includes several prefunded development accounts.
- 🧑‍⚖️ Development accounts are used as validators, collators, and `sudo` accounts.

## Runtime development

We recommend using [`chopsticks`](https://github.com/AcalaNetwork/chopsticks) when the focus is more on the runtime
development and `OmniNode` is enough as is.

### Install chopsticks

To use `chopsticks`, please install the latest version according to the installation [guide](https://github.com/AcalaNetwork/chopsticks?tab=readme-ov-file#install).

### Build a raw chain spec

Build the `parachain-template-runtime` as mentioned before in this guide and use `chain-spec-builder`
again but this time by passing `--raw-storage` flag:

```sh
chain-spec-builder create --raw-storage --relay-chain "rococo-local" --para-id {{PARACHAIN_ID}} --runtime \
    target/release/wbuild/parachain-template-runtime/parachain_template_runtime.wasm named-preset development
```

### Start `chopsticks` with the chain spec

```sh
npx @acala-network/chopsticks@latest --chain-spec <path/to/chain_spec.json>
```

### Alternatives

`OmniNode` can be still used for runtime development if using the `--dev` flag, while `parachain-template-node` doesn't
support it at this moment. It can still be used to test a runtime in a full setup where it is started alongside a
relay chain network (see [Parachain Template node](#parachain-template-node) setup).

## Contributing

We welcome contributions to Barcus Network! Here's how you can contribute:

1. **Fork the repository** - Create your own fork of the code
2. **Create a feature branch** - `git checkout -b feature/amazing-feature`
3. **Commit your changes** - `git commit -m 'Add some amazing feature'`
4. **Push to the branch** - `git push origin feature/amazing-feature`
5. **Open a Pull Request** - Submit your changes for review

### Development Guidelines

- Follow Rust best practices and conventions
- Write tests for new functionality
- Update documentation as needed
- Ensure all tests pass before submitting PRs

### Staying Updated with Polkadot SDK

This project is based on the [Polkadot SDK Parachain Template](https://github.com/paritytech/polkadot-sdk-parachain-template). To pull upstream updates:

```sh
# Fetch updates from the template
git fetch upstream

# Merge updates into your branch
git merge upstream/master
```

## Resources

### Barcus Network

- 📦 **Repository:** [github.com/biokeyper/barcus-network](https://github.com/biokeyper/barcus-network)
- 🐛 **Issues:** [Submit an issue](https://github.com/biokeyper/barcus-network/issues)
- 💬 **Discussions:** [GitHub Discussions](https://github.com/biokeyper/barcus-network/discussions)

### Polkadot & Substrate Resources

- 🧑‍🏫 **Polkadot Documentation:** [docs.polkadot.com](https://docs.polkadot.com/)
- 🧑‍🔧 **Polkadot SDK Docs:** [Polkadot SDK Documentation](https://github.com/paritytech/polkadot-sdk#-documentation)
- 📚 **Substrate Tutorials:** [docs.substrate.io](https://docs.substrate.io/)
- 💬 **Substrate StackExchange:** [substrate.stackexchange.com](https://substrate.stackexchange.com/)
- 👥 **Polkadot Discord:** [Official Polkadot Discord](https://polkadot-discord.w3f.tools/)
- 📱 **Telegram:** [Substrate Developers](https://t.me/substratedevs)

---

<div align="center">

**Built with ❤️ by [BioKeyper](https://github.com/biokeyper)**

*Powered by [Polkadot SDK](https://github.com/paritytech/polkadot-sdk)*

</div>

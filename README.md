# Alignment Protocol CLI

A unified command-line interface for interacting with the Alignment Protocol on Solana. This CLI combines both user and admin functionality in a single tool, with admin commands clearly marked with an [ADMIN] prefix.

The Alignment Protocol is a decentralized data quality validation system, enabling contributors to submit data and validators to vote on its quality. It uses a dual-token system with temporary and permanent alignment and reputation tokens.

## Installation

### Prerequisites

- [Solana CLI tools](https://docs.solana.com/cli/install-solana-cli-tools) installed and configured.

### Downloading the Binary

1. Go to the [Alignment Protocol CLI Releases Page](https://github.com/your-organization/alignment-protocol-cli/releases) (Replace with the actual URL).
2. Download the appropriate binary for your operating system (e.g., `alignment-protocol-cli-macos-amd64`, `alignment-protocol-cli-linux-amd64`, `alignment-protocol-cli-windows-amd64.exe`).
3. (Optional but recommended) Rename the downloaded file to `alignment-protocol-cli` (or `alignment-protocol-cli.exe` on Windows) and place it in a directory included in your system's PATH (e.g., `/usr/local/bin` or `~/bin` on Linux/macOS).

## Quick Start

1. Make sure you have a Solana keypair (default at `~/.config/solana/id.json`)
2. Set up your user profile (creates all required token accounts automatically):
   ```bash
   alignment-protocol-cli user create-profile
   ```
3. Initialize your balance for a specific topic (replace `0` with the actual topic ID):
   ```bash
   alignment-protocol-cli user initialize-topic-balance --topic-id 0
   ```
4. Submit data to a topic (replace `0` with the topic ID):
   ```bash
   alignment-protocol-cli submission submit 0 "Your data description or URI"
   ```
5. (Admin only) Create a new topic:
   ```bash
   alignment-protocol-cli topic create "Climate Data" "Repository for validated climate datasets"
   ```
6. (Admin only) Update token minting parameters:
   ```bash
   alignment-protocol-cli config update-tokens-to-mint 1000
   ```

## Command Reference

The CLI is organized into logical command groups that mirror the protocol's functionality. Admin commands are clearly marked with **[ADMIN]**.

### Main Command Groups

1. Topic - Topic management
   - List: View all topics in the protocol
   - View: View details of a specific topic
   - **[ADMIN]** Create: Create a new topic with name, description, custom voting phases
2. User - User account setup
   - CreateProfile: Create a user profile with all necessary token accounts
   - Profile: View user profile details and token balances
3. Submission - Data submission management
   - Submit: Submit data to a specific topic
   - Link: Link existing submission to another topic
   - Finalize: Finalize a submission after voting to handle token conversion
4. Vote - Voting operations
   - Commit: First phase of voting (commit a hidden vote)
   - Reveal: Second phase of voting (reveal previously committed vote)
   - Finalize: Finalize a vote to handle token conversion
   - **[ADMIN]** SetPhases: Set custom voting phase timestamps
5. Token - Token operations
   - Stake: Stake temp alignment tokens for a topic to earn reputation
   - **[ADMIN]** Mint: Mint tokens to a specific user
6. Query - Data query and exploration
   - State: View protocol state
   - Submission/Submissions: View specific or all submissions
   - SubmissionTopic: Check submission status in a specific topic
   - Vote: Check vote details
7. Debug - Debugging helpers
   - TokenAccount: Debug token account status
   - Tx: View detailed transaction logs
8. **[ADMIN]** Init - Protocol initialization
   - State: Initialize protocol state
   - TempAlignMint/AlignMint/TempRepMint/RepMint: Initialize specific token mints
   - All: Initialize all accounts at once
9. **[ADMIN]** Config - Protocol configuration
   - UpdateTokensToMint: Update number of tokens to mint per submission

### Global Options

- `--keypair <PATH>`: Path to your Solana keypair (default: ~/.config/solana/id.json)
- `--cluster <URL>`: Solana cluster to use (default: devnet)
- `--program-id <PUBKEY>`: Program ID for the Alignment Protocol

### Topic Management

```bash
# List all topics
alignment-protocol-cli topic list

# View a specific topic
alignment-protocol-cli topic view 0

# [ADMIN] Create a new topic (optionally specify durations)
alignment-protocol-cli topic create "Topic Name" "Topic Description" --commit-duration 86400 --reveal-duration 86400
```

### User Account Setup

```bash
# Create a user profile (all necessary token accounts are created automatically)
alignment-protocol-cli user create-profile

# Initialize user balance for a specific topic (required before submitting/voting on that topic)
alignment-protocol-cli user initialize-topic-balance --topic-id 0

# View user profile
alignment-protocol-cli user profile
alignment-protocol-cli user profile <PUBKEY>
```

#### What Happens During User Profile Creation

When you run `alignment-protocol-cli user create-profile`, the CLI performs several operations to set up everything you need to interact with the protocol:

1. **User Profile PDA**: Creates a program-derived address (PDA) that stores your on-chain profile information, including balances of topic-specific tokens and permanent reputation.

2. **Permanent Alignment Token Account**: Creates an Associated Token Account (ATA) linked to your wallet that can hold permanent Align tokens. These tokens are received when your submitted data is validated and accepted.

3. **Permanent Reputation Token Account**: Creates an ATA for permanent Rep tokens. These tokens are earned when you vote correctly on submitted data.

4. **Temporary Alignment Token Vault**: Creates a protocol-owned PDA (not an ATA) that holds temporary alignment tokens. This account is controlled by the protocol to ensure tokens can only be converted to permanent tokens when submissions are validated.

5. **Temporary Reputation Token Vault**: Creates a protocol-owned PDA for temporary reputation tokens. These tokens are staked during voting and can be converted to permanent tokens based on voting outcomes.

**Important**: After creating a profile, you must initialize your balance for each specific topic you intend to interact with using `user initialize-topic-balance --topic-id <ID>`. This sets up the necessary topic-specific accounts within your profile.

The CLI handles most of these steps, making it simpler to get started. Commands like `submission submit`, `vote commit`, and `token stake` automatically check whether you have a profile set up and the relevant topic balance initialized. If not, the CLI will show an error message directing you to create a profile or initialize the topic balance first.

### Data Submission

```bash
# Submit data to a topic (replace 0 with topic ID)
alignment-protocol-cli submission submit 0 "Your data description or IPFS URI like ipfs://QmHash"

# Link an existing submission to another topic
alignment-protocol-cli submission link <SUBMISSION_PUBKEY> <TARGET_TOPIC_ID>

# Finalize a submission after voting (replace 0 with topic ID)
alignment-protocol-cli submission finalize <SUBMISSION_PUBKEY> 0
```

### Voting

```bash
# Commit a vote (first phase) - replace <SUBMISSION_PUBKEY> and 0 with actual values
alignment-protocol-cli vote commit <SUBMISSION_PUBKEY> 0 yes 100 "secret-nonce"
# Optional flag: --permanent (if using permanent REP tokens)

# Reveal a vote (second phase)
alignment-protocol-cli vote reveal <SUBMISSION_PUBKEY> 0 yes "secret-nonce"

# Finalize a vote
alignment-protocol-cli vote finalize <SUBMISSION_PUBKEY> 0

# [ADMIN] Set voting phases (replace 0 with topic ID)
alignment-protocol-cli vote set-phases <SUBMISSION_PUBKEY> 0 --commit-start $(date +%s) --commit-end $(($(date +%s) + 3600)) --reveal-start $(date +%s) --reveal-end $(($(date +%s) + 3600))
```

### Token Operations

```bash
# Stake temporary alignment tokens for a topic (replace 0 with topic ID)
alignment-protocol-cli token stake 0 500

# [ADMIN] Mint tokens to a user
alignment-protocol-cli token mint temp-align Gn5Wz88RK2qCsJAPUyE9gThvFWjUTvXXYCdjfvJZk5Ge 1000
```

### Protocol Initialization (Admin)

```bash
# [ADMIN] Initialize protocol state
alignment-protocol-cli init state

# [ADMIN] Initialize all accounts (state and all token mints)
alignment-protocol-cli init all --oracle-pubkey <ORACLE_PUBKEY>
```

### Protocol Configuration (Admin)

```bash
# [ADMIN] Update tokens to mint per submission
alignment-protocol-cli config update-tokens-to-mint 1000
```

### Querying Data

```bash
# Query protocol state
alignment-protocol-cli query state

# Query a specific submission by its pubkey
alignment-protocol-cli query submission <SUBMISSION_PUBKEY>

# Query all submissions
alignment-protocol-cli query submissions
alignment-protocol-cli query submissions --by <USER_PUBKEY> --topic 0

# Query submission status in a specific topic
alignment-protocol-cli query submission-topic <SUBMISSION_PUBKEY> 0

# Query vote information for a specific validator on a submission
alignment-protocol-cli query vote <SUBMISSION_PUBKEY> 0 <VALIDATOR_PUBKEY>

# Query topic-specific token balance for the current user (or specified user)
alignment-protocol-cli query topic-balance 0
alignment-protocol-cli query topic-balance 0 --user <USER_PUBKEY>
```

### Debugging

```bash
# Debug token account status
alignment-protocol-cli debug token-account temp-align
alignment-protocol-cli debug token-account align <USER_PUBKEY>

# Get transaction logs
alignment-protocol-cli debug tx <TX_SIGNATURE>
```

## Token System

The protocol uses four types of tokens:

1. **tempAlign** (temporary alignment tokens): Given to data contributors
2. **Align** (permanent alignment tokens): Converted from tempAlign when data is accepted
3. **tempRep** (temporary reputation tokens): Earned by validators for specific topics
4. **Rep** (permanent reputation tokens): Converted from tempRep for correct votes

## Protocol Workflow

1. **Protocol Deployment**: Protocol is deployed and initialized by administrators
2. **Topic Creation**: Create topics for data submissions
3. **User Setup**: Create user profile with all necessary token accounts
4. **Initialize Topic Balance**: Users initialize their balance for specific topics they want to interact with.
5. **Data Submission**: Contributors submit data to topics and receive tempAlign tokens
6. **Voting**:
   - **Commit Phase**: Validators commit hidden votes using a hash
   - **Reveal Phase**: Validators reveal their votes with the original data
7. **Submission Finalization**:
   - If accepted, contributor's tempAlign tokens convert to permanent Align tokens
   - If rejected, tempAlign tokens remain locked
8. **Vote Finalization**:
   - Correct votes: Validator's tempRep tokens convert to permanent Rep tokens
   - Incorrect votes: Validator's tempRep tokens are burned

## Advanced Usage

### Creating a Commit Hash Manually

The CLI automatically generates the commit hash for you, but if you need to create it manually:

```bash
# Format: SHA-256(validator_pubkey + submission_topic_link_pubkey + vote_choice + nonce)
# Where vote_choice is "yes" or "no"
```

### Testing the Protocol

For testing, you can use the `vote set-phases` command to set arbitrary timestamps for voting phases:

```bash
# Set all phases to be active now for testing
alignment-protocol-cli vote set-phases <SUBMISSION_PUBKEY> 0 --commit-start $(date +%s) --commit-end $(($(date +%s) + 3600)) --reveal-start $(date +%s) --reveal-end $(($(date +%s) + 3600))
```

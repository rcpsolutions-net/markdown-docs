# Walkthrough: Dedicated Direct Deposit Command Session for gs-cli

Implemented a dedicated command group `direct-deposit` (alias: `dd`) in [gs-cli](file:///home/lham/dev/cli/gs-cli) to support GET, PUT (update/overwrite/clear), and DELETE operations on employee direct deposit settings according to the Greenshades OpenAPI specs.

## Changes Made

### 1. Created Command Module
[commands/direct-deposit.ts](file:///home/lham/dev/cli/gs-cli/commands/direct-deposit.ts)

Implemented:
- **`get|pull <employeeId>`**:
  - GET `/employees/{employeeId}/directdeposit`
  - Format output as formatted table (`--output table`, default) or JSON (`--output json`).
  - Handles empty accounts list gracefully.
- **`update|put|set <employeeId>`**:
  - PUT `/employees/{employeeId}/directdeposit`
  - Overwrites existing direct deposit settings or clears accounts.
  - Supports multiple input modes:
    - `--clear`: Clears all direct deposit settings by sending an empty array `[]`.
    - `-f, --file <filePath>`: Reads and parses a JSON array from a file.
    - `-d, --data <jsonData>`: Accepts inline JSON string array.
    - CLI flags: `--routing`, `--account`, `--type` (`Checking` | `Savings`), `--amount`, `--percent`, `--remainder`, `--prenote`, `--paycard-type`.
    - Interactive mode: If no file, data, or account flags are passed, launches an interactive `inquirer` prompt.
  - Client-side validation:
    - 9-digit routing number (`^\d{9}$`)
    - Max 25 characters account number
    - Account type (`Checking` or `Savings`)
    - Ensures at most one remainder designation across entries.
- **`delete|clear|del <employeeId>`**:
  - DELETE `/employees/{employeeId}/directdeposit`
  - Deletes all direct deposit accounts.
  - Includes confirmation prompt (bypassable with `-y` or `--force`).

### 2. Registered in Main CLI Entrypoint
[index.js](file:///home/lham/dev/cli/gs-cli/index.js)

- Imported `createDirectDepositCommands` and added it to the Commander program:
  ```javascript
  program.addCommand(createDirectDepositCommands());
  ```

---

## Verification & Usage Examples

### 1. Help Commands
```bash
greenshades direct-deposit --help
# or shorthand:
greenshades dd --help
```

### 2. Get Direct Deposit Settings
```bash
# Table output (default)
greenshades dd get <employeeId>

# Raw JSON output
greenshades dd get <employeeId> -o json
```

### 3. Update / Overwrite Direct Deposit Settings

```bash
# Using command-line flags for a primary remainder account:
greenshades dd update <employeeId> --routing 123456789 --account 987654321 --type Checking --remainder

# Using a fixed dollar split:
greenshades dd update <employeeId> --routing 123456789 --account 987654321 --type Checking --amount 250.00

# Using a JSON file:
greenshades dd update <employeeId> -f ./accounts.json

# Interactive mode:
greenshades dd update <employeeId>

# Clear all direct deposit accounts using PUT:
greenshades dd update <employeeId> --clear
```

### 4. Delete Direct Deposit Settings

```bash
# Interactive confirmation prompt:
greenshades dd delete <employeeId>

# Force delete without confirmation:
greenshades dd delete <employeeId> --force
```

### 5. Automated Validation Tests Run
- Verified `node bin/greenshades.js --help` lists `direct-deposit|dd`.
- Verified `node bin/greenshades.js dd update --help` and `dd delete --help`.
- Verified client-side rejection of malformed routing numbers (`123` rejected before network call).
- Verified strict validation of account types (`InvalidType` rejected with informative error).
- Verified automatic authentication token refresh against live Greenshades auth.

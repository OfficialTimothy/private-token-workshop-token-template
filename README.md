

# 🪙 Official Timothy Ufot Compliant Private Token (Aleo Workshop Project)

The **Official Timothy Ufot Compliant Private Token** is a privacy-preserving smart contract built on the **Aleo blockchain** as part of the **Aleo Workshop**.
It demonstrates how to build a **regulatory-compliant token** that integrates **OFAC (Office of Foreign Assets Control)** address verification for compliant minting and transferring of tokens, both **publicly** and **privately**.

---

## 🧾 Deployment Information

* **Program Name:** `official_timothy_ufot_token.aleo`
* **Registry Module:** `workshop_ofac.aleo` (for compliance verification)
* **Blockchain:** Aleo Testnet
* **Technology Stack:** Leo Language + Aleo zkVM
* **Author:** Official Timothy Ufot

---

## 📜 Deployment Details

* **🆔 Deployment ID:** `at1yge0vf447kdqpdp4ungj4cspxxs76efkw02n3lsv7zrxkzv0vqrsnyl5k5`
* **⚙️ Execution ID (Transaction ID):** `at1j9wdkv8e8jakrwdy5lg3y5rxvlmrq8s4wn0xhszmf9ewrdehsgxsrf384s`
* **🔗 View on Aleo Testnet:** [Aleo Explorer](https://testnet.aleo.info/program/official_timothy_ufot_token.aleo)
* **📦 Program Name:** `official_timothy_ufot_token.aleo`

---

## ⚙️ Key Features and Functionalities

### **1. OFAC-Compliant Token Minting**

* Integrates `workshop_ofac.aleo` to ensure that **only non-sanctioned addresses** can receive or hold tokens.
* Supports two minting types:

  * **Public Mint:** Tokens are minted to a public address on-chain.
  * **Private Mint:** Tokens are minted as **private records**, enhancing user privacy.

### **2. Public Token Transfers**

* Transfers tokens between public addresses with **OFAC validation**.
* Prevents sanctioned wallets from participating in transfers.
* Updates balances on-chain in a **mapping structure** (`balances`).

### **3. Private Token Transfers**

* (To be extended): Enables private token-to-token transfers where both sender and recipient are verified for compliance.
* Uses **record-based tokenization**, ensuring that ownership and balance remain private.

### **4. Asynchronous Compliance Checks**

* Uses Aleo’s `Future` and `await()` mechanisms for asynchronous validation of addresses via `workshop_ofac.aleo/address_check`.
* Ensures compliance **before completing on-chain mint or transfer**.

### **5. Secure Record Management**

Defines a `Token` record structure:

```leo
record Token {
    owner: address,
    amount: u64
}
```

Records are cryptographically linked to the owner, ensuring **confidential and verifiable ownership**.

---

## 🧠 Smart Contract Overview

### **Contract Structure**

```leo
program official_timothy_ufot_token.aleo {
    mapping balances: address => u64;

    record Token {
        owner: address,
        amount: u64,
    }

    @noupgrade
    async constructor() {}
}
```

### **Compliance Integration**

Each public and private function calls:

```leo
let address_check: Future = workshop_ofac.aleo/address_check(recipient);
```

This ensures that **all addresses are screened** against the OFAC list before tokens are minted or transferred.

---

## 💰 Core Functions

### **1. Mint Public Tokens**

**Command:**

```sh
leo run mint_public <recipient_address> <amount>
```

**Purpose:**
Mints tokens directly to a public address after verifying compliance with `workshop_ofac.aleo`.

**Key Logic:**

```leo
async transition mint_public(public recipient: address, public amount: u64) -> Future {
    let address_check: Future = workshop_ofac.aleo/address_check(recipient);
    return mint_public_onchain(recipient, amount, address_check);
}

async function mint_public_onchain(public recipient: address, public amount: u64, public address_check: Future) {
    address_check.await();
    let current_amount: u64 = Mapping::get_or_use(balances, recipient, 0u64);
    Mapping::set(balances, recipient, current_amount + amount);
}
```

---

### **2. Mint Private Tokens**

**Command:**

```sh
leo run mint_private <recipient_address> <amount>
```

**Purpose:**
Mints tokens privately to a recipient’s wallet, producing a confidential `Token` record.
Also validates address compliance before record creation.

**Key Logic:**

```leo
async transition mint_private(private recipient: address, private amount: u64) -> (Token, Future) {
    let address_check: Future = workshop_ofac.aleo/address_check(recipient);
    let token: Token = Token { owner: recipient, amount: amount };
    return (token, mint_private_onchain(address_check));
}

async function mint_private_onchain(address_check: Future) {
    address_check.await();
}
```

---

### **3. Transfer Public Tokens**

**Command:**

```sh
leo run transfer_public <recipient_address> <amount>
```

**Purpose:**
Transfers tokens publicly between two Aleo accounts, validating the recipient through the OFAC module.

**Key Logic:**

```leo
async transition transfer_public(public recipient: address, public amount: u64) -> Future {
    let address_check: Future = workshop_ofac.aleo/address_check(recipient);
    return transfer_public_onchain(self.signer, recipient, amount, address_check);
}

async function transfer_public_onchain(public sender: address, public recipient: address, public amount: u64, public address_check: Future) {
    address_check.await();
    let sender_amount: u64 = Mapping::get_or_use(balances, sender, 0u64);
    Mapping::set(balances, sender, sender_amount - amount);
    let receiver_amount: u64 = Mapping::get_or_use(balances, recipient, 0u64);
    Mapping::set(balances, recipient, receiver_amount + amount);
}
```

---

### **4. Transfer Private Tokens (Optional Extension)**

**Purpose:**
Enables confidential token transfers using private `Token` records while still adhering to compliance requirements.

**Concept:**

* Consumes the sender’s private record.
* Generates a **recipient record** (transferred amount) and **sender record** (remaining balance).
* Ensures compliance by calling `address_check` asynchronously before finalizing the transaction.

---

## 🧩 Technical Highlights

| Feature              | Description                                                    |
| -------------------- | -------------------------------------------------------------- |
| **Language**         | Leo                                                            |
| **Mapping**          | Tracks balances of public addresses                            |
| **Records**          | Stores private token ownership data                            |
| **Async Validation** | Ensures address compliance before processing                   |
| **Extensible**       | Can be extended for `burn`, `approve`, and `private transfers` |

---

## 🧱 Use Case Scenarios

### ✅ **Regulated Token Deployments**

Ensures that **minting and transfers** are restricted to verified users, complying with global AML/OFAC standards.

### 🔐 **Privacy-Preserving Transfers**

Enables confidential transactions that preserve user privacy while enforcing regulatory checks.

### 🏛️ **Enterprise and Government Use**

Ideal for building **compliant stablecoins, CBDCs**, or **corporate token systems** that require verifiable compliance.

-

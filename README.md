# Canto audit details
- Total Prize Pool: $13,500
  - HM awards: $9,900
  - Analysis awards: $500
  - QA awards: $300
  - Gas awards: $300
  - Judge awards: $2,000
  - Scout awards: $500
 
  ❗️Awarding Note for Wardens, Judges, and Lookouts: If you want to claim your awards in $ worth of CANTO, you must follow the steps outlined in this thread; otherwise you'll be paid out in USDC.
 
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2024-03-canto/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts March 29, 2024 20:00 UTC
- Ends April 3, 2024 20:00 UTC

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-03-canto/blob/main/4naly3er-report.md).

_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._


# Overview

# Application Specific Dollar Omnichain Fungible Token (asD-OFT)

## Background

asD allows for protocols to earn yield on their users' dollar deposits. asD is always pegged to 1 $NOTE and this $NOTE can be deposited into the Canto Lending Market to accrue interest. This version of asD implements LayerZero's Omnichain Fungible Token to allow creators from any LayerZero supported chain to earn extra revenue from all stable coin deposits.

#### Useful background information

- NOTE: https://docs.canto.io/overview/canto-unit-of-account-usdnote
- Canto Lending Market: https://docs.canto.io/overview/canto-lending-market-clm
- Compound cTOKEN Documentation: https://docs.compound.finance/v2/ctokens
- Layer Zero: https://docs.layerzero.network/contracts/overview
- asD ERC20 c4 audit: https://code4rena.com/audits/2023-11-canto-application-specific-dollars-and-bonding-curves-for-1155s

## asD Flow

The purpose of implementing cross chain functionality with asD is to allow developers from any supported chain to earn Canto Lending Market yield without needing to directly deploy their entire protocol on the Canto application layer. By using Layer Zero v2, we can accomplish this by bridging whitelisted stable coins to mint asD tokens that can then be bridged back to the original application.

![asD Flow](asdFlow.png)

## Contracts

### ASDOFT

The ASDOFT contract is a direct implementation of the ASD ERC20 token used for [1155Tech](https://github.com/code-423n4/2023-11-canto) however it also implements LayerZero v2 OFT functionality. This allows for asD tokens minted on Canto as well as easily bridged back to the source application to be used in the protocol.

In order to ensure a 1:1 peg to $NOTE, only the ASDOFT should be deployed on Canto for minting, and regular OFT implementations should be deployed on all other supported chains.

### ASDRouter

The ASDRouter contract allows for the minting of asD tokens to be done in a single transaction originating from any supported chain. A stable coin represented as an OFT is sent to the ASDRouter with "lzCompose" options passed to tell the Layer Zero executor to call `lzCompose` once it receives the message.

```javascript
 function lzCompose(
    address _from,
    bytes32 _guid,
    bytes calldata _message,
    address _executor,
    bytes calldata _extraData) external payable;
```

A user who wants to obtain asD tokens must send their stable assets to the ASDRouter through Layer Zero. The `_message` must be formatted as bytes with the form of the OFTComposeMessage struct:

```javascript
struct OftComposeMessage {
    uint32 _dstLzEid; // destination endpoint id
    address _dstReceiver; // receiver on destination
    address _dstAsdAddress; // asD address on destination
    address _cantoAsdAddress; // asD on Canto (where to mint asD from)
    uint256 _minAmountASD; // minimum amount (slippage for swap)
    address _cantoRefundAddress; // canto refund address
    uint256 _feeForSend; // fee for bridge to destination chain
}
```

This struct tells the Router which asD tokens to mint and where to send them after they are obtained. The `lzCompose` function on the Router performs the following steps

1. Deposits stable coin for asdUSDC
2. Swaps asdUSDC in ambient pool for $NOTE
3. Deposits $NOTE to asD Vault for asD tokens
4. Sends asD tokens to destination chain and address

After lzCompose has been called, the Router should never have any tokens left over (all tokens are refunded or sent). With Layer Zero execution, if the `lzCompose` reverts, the OFT tokens are still received by the Router, but the compose message can be retried. Because of this, the Router protects against reverts by implementing saftey checks and refunds during each step. All refunds will be sent to the user's address on Canto. For any message that contains `msg.value`, the entire amount will also be refunded to the user as Canto.

Saftey checks:

1. Incorrect compose message formatting (OFT refunded)
2. Not whitelisted stable coin version (OFT refunded)
3. Unsuccessful swap to $NOTE (usdcOFT refunded)
4. Unsuccessful deposit into asD vault ($NOTE refunded)
5. Unsuccessful bridge of asD to destination of insufficient fee (asD refunded)

### ASDUSDC

ASDUSDC is a simple wrapper for all of the different representations for the same stable coins that might be bridged through Layer Zero. The purpose of this is to limit the amount of pools needed for swapping tokens to $NOTE. With this wrapper, only one pool is needed and all OFT's representing the same can be wrapped into ASDUSDC and swapped for Note easily. A user can deposit any of the whitelisted usdc versions to mint ASDUSDC as well as withdraw any of those same versions (as long as there is enough locked) by burning their ASDUSDC.


## Links

- **Previous audits:** None
- **Documentation:** https://github.com/Plex-Engineer/ASD-V2/blob/main/README.md
- **Website:** https://canto.io/
- **X/Twitter:** https://twitter.com/CantoPublic
- **Discord:** https://discord.com/invite/63GmEXZsVf

---

# Scope

### Files in scope


| File                                  | Logic Contracts | Interfaces |  SLOC   | Purpose | Libraries used |
|:------------------------------------- |:---------------:|:----------:|:-------:|:------- |:-------------- |
| /contracts/clm/CTokenInterfaces.sol   |      ****       |     2      |   12    |    Interaction with Compound Lending Market cTokens     |       ****         |
| /contracts/clm/Turnstile.sol          |      ****       |     1      |    3    |   Registers contract to CSR (contract secured revenue)      |      ****          |
| /contracts/asd/asdOFT.sol             |        1        |    ****    |   47    |    Mints and burns ASD tokens by depositing/withdrawing $NOTE and supplying/withdrawing from Canto Lending Market     |     ERC20, OFT, CErc20Interface           |
| /contracts/asd/asdRouter.sol          |        1        |    ****    |   138   |   Routes stable OFTs to correct ASD vault and returns to user on destination chain in a single transaction      |    OptionsBuilder, IOFT, OFTComposeMsgCodec, IOAppComposer            |
| /contracts/asd/asdUSDC.sol            |        1        |    ****    |   40    |   Wrapper for different OFT representations of the same underlying token to be used in dex      |     ERC20           |
| /contracts/ambient/CrocInterfaces.sol |      ****       |     2      |    7    |   Interface for ambient dex for router to swap for Note      |      ****          |
| **Totals**                            |      **3**      |   **5**    | **247** |         |                |

### Files out of scope
| File |
| ---- |
| /contracts/test-contracts/USDCOFT.sol |
| /contracts/test-contracts/TestERC20.sol |
| /contracts/test-contracts/TestASD.sol |
| /contracts/test-contracts/MockCrocSwap.sol |
| /contracts/asd/OFT.sol |
| /contracts/asd/OFTAdapter.sol |

## Scoping Q &amp; A

### General questions


| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |      Layer Zero OFT, USDC            |
| Test coverage                           |  75%                          |
| ERC721 used  by the protocol            |            None              |
| ERC777 used by the protocol             |           None                |
| ERC1155 used by the protocol            |              None            |
| Chains the protocol will be deployed on | Ethereum, Arbitrum, Polygon, OtherCanto  |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- |:------:|
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |   ❌   |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |   ✅   |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) |   ❌   |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |   ❌   |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              |   ❌   |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      |   ❌   |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              |   ❌   |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            |   ❌   |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    |   ❌   |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    |   ✅   |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    |   ✅   |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  |   ❌   |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |   ✅   |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          |   ✅   |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |   ✅   |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              |   ✅   |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                |   ❌   |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- |:------:|
| Enabling/disabling fees (e.g. Blur disables/enables fees) |   No   |
| Pausability (e.g. Uniswap pool gets paused)               |   No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No   |


### EIP compliance checklist
None


# Additional context

## Main invariants

1. `ASDRouter` native balance should always be zero.
2. `ASDRouter` balance of any token should always be zero after lzCompose has completed
3. `ASDUSDC` usdcBalances mapping should always sum to the totalSupply


## Attack ideas (where to focus for bugs)
* Since Layer Zero executors call `lzCompose` after sending funds to the `ASDRouter` Contract, this `lzCompose` function should never revert. 
* All external calls should be checked for success and decoding bytes must never fail. 
* All errors should result in refunds so that tokens do not get trapped in Layer Zero Protocol.


## All trusted roles in the protocol


| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| ASDUSDC owner                          | Sets whitelist for tokens             |


## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

None



## Running tests

```bash
git clone https://github.com/code-423n4/2024-03-canto.git

git submodule update --init --recursive

npm install

npx hardhat compile

npx hardhat test
```
To run code coverage
```bash
npx hardhat coverage
```
To run gas benchmarks
```bash
REPORT_GAS=true npx hardhat test
```
![canto_gas_report](https://github.com/code-423n4/2024-03-canto/assets/65364747/e029a8e9-3375-4fa8-8b83-531c916dc138)
![canto_coverage](https://github.com/code-423n4/2024-03-canto/assets/65364747/dd43c395-56fc-45b4-94c4-2e014b0191f8)


## Miscellaneous
Employees of Canto and employees' family members are ineligible to participate in this audit.

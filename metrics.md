
NB: This report has been created using [Solidity-Metrics](https://github.com/Consensys/solidity-metrics)
<sup>

# Solidity Metrics for Scoping for Plex-Engineer - ASD-V2

## Table of contents

- [Scope](#t-scope)
    - [Source Units in Scope](#t-source-Units-in-Scope)
    - [Out of Scope](#t-out-of-scope)
        - [Excluded Source Units](#t-out-of-scope-excluded-source-units)
        - [Duplicate Source Units](#t-out-of-scope-duplicate-source-units)
        - [Doppelganger Contracts](#t-out-of-scope-doppelganger-contracts)
- [Report Overview](#t-report)
    - [Risk Summary](#t-risk)
    - [Source Lines](#t-source-lines)
    - [Inline Documentation](#t-inline-documentation)
    - [Components](#t-components)
    - [Exposed Functions](#t-exposed-functions)
    - [StateVariables](#t-statevariables)
    - [Capabilities](#t-capabilities)
    - [Dependencies](#t-package-imports)
    - [Totals](#t-totals)

## <span id=t-scope>Scope</span>

This section lists files that are in scope for the metrics report.

- **Project:** `Scoping for Plex-Engineer - ASD-V2`
- **Included Files:** 
8
- **Excluded Files:** 
4
- **Project analysed:** `https://github.com/Plex-Engineer/ASD-V2` (`@a54c81df484db5d39f3335434d4986796b5c3686`)

### <span id=t-source-Units-in-Scope>Source Units in Scope</span>

Source Units Analyzed: **`8`**<br>
Source Units in Scope: **`8`** (**100%**)

| Type | File   | Logic Contracts | Interfaces | Lines | nLines | SLOC | Comment Lines | Complex. Score | Capabilities |
| ---- | ------ | --------------- | ---------- | ----- | ------ | ----- | ------------- | -------------- | ------------ |
| 🔍 | /contracts/clm/CTokenInterfaces.sol | **** | 2 | 19 | 3 | 12 | **** | 18 | **** |
| 🔍 | /contracts/clm/Turnstile.sol | **** | 1 | 3 | 2 | 3 | **** | 3 | **** |
| 📝 | /contracts/asd/OFT.sol | 1 | **** | 7 | 7 | 5 | **** | 4 | **** |
| 📝 | /contracts/asd/OFTAdapter.sol | 1 | **** | 7 | 7 | 5 | **** | 4 | **** |
| 📝 | /contracts/asd/asdOFT.sol | 1 | **** | 79 | 79 | 47 | 29 | 45 | **** |
| 📝 | /contracts/asd/asdRouter.sol | 1 | **** | 263 | 263 | 138 | 93 | 94 | **<abbr title='Payable Functions'>💰</abbr><abbr title='Initiates ETH Value Transfer'>📤</abbr><abbr title='Uses Hash-Functions'>🧮</abbr>** |
| 📝 | /contracts/asd/asdUSDC.sol | 1 | **** | 82 | 82 | 40 | 34 | 43 | **** |
| 🔍 | /contracts/ambient/CrocInterfaces.sol | **** | 2 | 9 | 3 | 7 | **** | 11 | **<abbr title='Payable Functions'>💰</abbr>** |
| 📝🔍 | **Totals** | **5** | **5** | **469**  | **446** | **257** | **156** | **222** | **<abbr title='Payable Functions'>💰</abbr><abbr title='Initiates ETH Value Transfer'>📤</abbr><abbr title='Uses Hash-Functions'>🧮</abbr>** |

##### <span>Legend</span>
<ul>
<li> <b>Lines</b>: total lines of the source unit </li>
<li> <b>nLines</b>: normalized lines of the source unit (e.g. normalizes functions spanning multiple lines) </li>
<li> <b>SLOC</b>: source lines of code</li>
<li> <b>Comment Lines</b>: lines containing single or block comments </li>
<li> <b>Complexity Score</b>: a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...) </li>
</ul>

### <span id=t-out-of-scope>Out of Scope</span>

### <span id=t-out-of-scope-excluded-source-units>Excluded Source Units</span>
Source Units Excluded: **`4`**

| File |
| ---- |
| /contracts/test-contracts/USDCOFT.sol |
| /contracts/test-contracts/TestERC20.sol |
| /contracts/test-contracts/TestASD.sol |
| /contracts/test-contracts/MockCrocSwap.sol |

## <span id=t-report>Report</span>

## Overview

The analysis finished with **`0`** errors and **`0`** duplicate files.





### <span style="font-weight: bold" id=t-inline-documentation>Inline Documentation</span>

- **Comment-to-Source Ratio:** On average there are`1.65` code lines per comment (lower=better).
- **ToDo's:** `0`

### <span style="font-weight: bold" id=t-components>Components</span>

| 📝Contracts   | 📚Libraries | 🔍Interfaces | 🎨Abstract |
| ------------- | ----------- | ------------ | ---------- |
| 5 | 0  | 5  | 0 |

### <span style="font-weight: bold" id=t-exposed-functions>Exposed Functions</span>

This section lists functions that are explicitly declared public or payable. Please note that getter methods for public stateVars are not included.

| 🌐Public   | 💰Payable |
| ---------- | --------- |
| 20 | 2  |

| External   | Internal | Private | Pure | View |
| ---------- | -------- | ------- | ---- | ---- |
| 20 | 28  | 0 | 1 | 6 |

### <span style="font-weight: bold" id=t-statevariables>StateVariables</span>

| Total      | 🌐Public  |
| ---------- | --------- |
| 9  | 9 |

### <span style="font-weight: bold" id=t-capabilities>Capabilities</span>

| Solidity Versions observed | 🧪 Experimental Features | 💰 Can Receive Funds | 🖥 Uses Assembly | 💣 Has Destroyable Contracts |
| -------------------------- | ------------------------ | -------------------- | ---------------- | ---------------------------- |
| `^0.8.22` |  | `yes` | **** | **** |

| 📤 Transfers ETH | ⚡ Low-Level Calls | 👥 DelegateCall | 🧮 Uses Hash Functions | 🔖 ECRecover | 🌀 New/Create/Create2 |
| ---------------- | ----------------- | --------------- | ---------------------- | ------------ | --------------------- |
| `yes` | **** | **** | `yes` | **** | **** |

| ♻️ TryCatch | Σ Unchecked |
| ---------- | ----------- |
| **** | **** |

### <span style="font-weight: bold" id=t-package-imports>Dependencies / External Imports</span>

| Dependency / Import Path | Count  |
| ------------------------ | ------ |
| @layerzerolabs/lz-evm-oapp-v2/contracts/oapp/interfaces/IOAppComposer.sol | 1 |
| @layerzerolabs/lz-evm-oapp-v2/contracts/oapp/libs/OptionsBuilder.sol | 1 |
| @layerzerolabs/lz-evm-oapp-v2/contracts/oft/OFT.sol | 2 |
| @layerzerolabs/lz-evm-oapp-v2/contracts/oft/OFTAdapter.sol | 1 |
| @layerzerolabs/lz-evm-oapp-v2/contracts/oft/interfaces/IOFT.sol | 1 |
| @layerzerolabs/lz-evm-oapp-v2/contracts/oft/libs/OFTComposeMsgCodec.sol | 1 |
| @openzeppelin/contracts/access/Ownable.sol | 2 |
| @openzeppelin/contracts/token/ERC20/ERC20.sol | 2 |
| @openzeppelin/contracts/token/ERC20/IERC20.sol | 1 |
| @openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol | 2 |


##### Contract Summary

 Sūrya's Description Report

 Files Description Table


|  File Name  |  SHA-1 Hash  |
|-------------|--------------|
| /contracts/clm/CTokenInterfaces.sol | e0e51fd3ee65b27d301f31549e3b68a9e256ed82 |
| /contracts/clm/Turnstile.sol | 197c8015e5c03ad171fcb3b56587d620a49b92a6 |
| /contracts/asd/OFT.sol | 6be8b81907046d3a1ff9d6756c8c2da4354382ff |
| /contracts/asd/OFTAdapter.sol | 39b78988ca975d2d179995bee297948279707902 |
| /contracts/asd/asdOFT.sol | abe6ae5f17ccfc465982570a47d2fdea13425ae8 |
| /contracts/asd/asdRouter.sol | bc0e32b5042abf6e376746cff202f8286e2326aa |
| /contracts/asd/asdUSDC.sol | f31f98d3dd2561c05916c2a8a0f4d118c348a0ae |
| /contracts/ambient/CrocInterfaces.sol | 4eeb4944d77069490d96219fdca44fb3ad75d881 |


 Contracts Description Table


|  Contract  |         Type        |       Bases      |                  |                 |
|:----------:|:-------------------:|:----------------:|:----------------:|:---------------:|
|     └      |  **Function Name**  |  **Visibility**  |  **Mutability**  |  **Modifiers**  |
||||||
| **CTokenInterface** | Interface |  |||
| └ | mint | External ❗️ | 🛑  |NO❗️ |
| └ | redeem | External ❗️ | 🛑  |NO❗️ |
| └ | redeemUnderlying | External ❗️ | 🛑  |NO❗️ |
| └ | exchangeRateStored | External ❗️ |   |NO❗️ |
| └ | balanceOfUnderlying | External ❗️ |   |NO❗️ |
||||||
| **CErc20Interface** | Interface |  |||
| └ | underlying | External ❗️ |   |NO❗️ |
| └ | mint | External ❗️ | 🛑  |NO❗️ |
| └ | redeemUnderlying | External ❗️ | 🛑  |NO❗️ |
||||||
| **Turnstile** | Interface |  |||
| └ | register | External ❗️ | 🛑  |NO❗️ |
||||||
| **OFT** | Implementation | LZOFT |||
| └ | <Constructor> | Public ❗️ | 🛑  | LZOFT |
||||||
| **OFTAdapter** | Implementation | LZAdapter |||
| └ | <Constructor> | Public ❗️ | 🛑  | LZAdapter |
||||||
| **ASDOFT** | Implementation | OFT |||
| └ | <Constructor> | Public ❗️ | 🛑  | OFT |
| └ | mint | External ❗️ | 🛑  |NO❗️ |
| └ | burn | External ❗️ | 🛑  |NO❗️ |
| └ | withdrawCarry | External ❗️ | 🛑  | onlyOwner |
||||||
| **ASDRouter** | Implementation | IOAppComposer, Ownable |||
| └ | <Constructor> | Public ❗️ | 🛑  |NO❗️ |
| └ | lzCompose | External ❗️ |  💵 |NO❗️ |
| └ | _decodeOFTComposeMsg | Internal 🔒 |   | |
| └ | _sendASD | Internal 🔒 | 🛑  | |
| └ | _refundToken | Internal 🔒 | 🛑  | |
| └ | _depositNoteToASDVault | Internal 🔒 | 🛑  | |
| └ | _swapOFTForNote | Internal 🔒 | 🛑  | |
| └ | ambientPoolFor | Internal 🔒 |   | |
||||||
| **ASDUSDC** | Implementation | ERC20, Ownable |||
| └ | <Constructor> | Public ❗️ | 🛑  | ERC20 |
| └ | updateWhitelist | External ❗️ | 🛑  | onlyOwner |
| └ | deposit | External ❗️ | 🛑  |NO❗️ |
| └ | withdraw | External ❗️ | 🛑  |NO❗️ |
| └ | recover | External ❗️ | 🛑  | onlyOwner |
||||||
| **ICrocSwapDex** | Interface |  |||
| └ | swap | External ❗️ |  💵 |NO❗️ |
| └ | readSlot | External ❗️ |   |NO❗️ |
||||||
| **ICrocImpact** | Interface |  |||
| └ | calcImpact | External ❗️ |   |NO❗️ |


 Legend

|  Symbol  |  Meaning  |
|:--------:|-----------|
|    🛑    | Function can modify state |
|    💵    | Function is payable |

____


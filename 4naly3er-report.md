# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings) | 2 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 1 |
| [GAS-3](#GAS-3) | For Operations that will not overflow, you could use unchecked | 16 |
| [GAS-4](#GAS-4) | Use Custom Errors instead of Revert Strings to save Gas | 7 |
| [GAS-5](#GAS-5) | Avoid contract existence checks by using low level calls | 1 |
| [GAS-6](#GAS-6) | Functions guaranteed to revert when called by normal users can be marked `payable` | 3 |
| [GAS-7](#GAS-7) | Using `private` rather than `public` for constants, saves gas | 1 |
| [GAS-8](#GAS-8) | Use of `this` instead of marking as `public` an `external` function | 3 |
| [GAS-9](#GAS-9) | Use != 0 instead of > 0 for unsigned integer comparison | 1 |
### <a name="GAS-1"></a>[GAS-1] `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings)
This saves **16 gas per instance.**

*Instances (2)*:
```solidity
File: contracts/asd/asdUSDC.sol

38:         usdcBalances[_usdcVersion] += _amount;

76:         usdcBalances[_usdcVersion] += amountToRecover;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
```solidity
File: contracts/asd/asdUSDC.sol

12:     mapping(address => bool) public whitelistedUSDCVersions;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-3"></a>[GAS-3] For Operations that will not overflow, you could use unchecked

*Instances (16)*:
```solidity
File: contracts/asd/asdOFT.sol

13:     address public immutable cNote; // Reference to the cNOTE token

54:         uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)

55:         require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying

65:         uint256 maximumWithdrawable = CTokenInterface(cNote).balanceOfUnderlying(address(this)) - totalSupply();

74:         require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

234:         int minAmountInt = int(_minAmountNote); // stack too deep fix

236:         if (isNoteBase && -baseFlow < minAmountInt) {

239:         } else if (!isNoteBase && -quoteFlow < minAmountInt) {

254:         return (uint128(-1 * (isNoteBase ? baseUsed : quoteUsed)), true);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

38:         usdcBalances[_usdcVersion] += _amount;

40:         uint256 amountToMint = _amount * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

58:         uint256 amountToWithdraw = _amount / (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

62:         usdcBalances[_usdcVersion] -= amountToWithdraw;

75:         uint amountToRecover = ERC20(_usdcVersion).balanceOf(address(this)) - usdcBalances[_usdcVersion];

76:         usdcBalances[_usdcVersion] += amountToRecover;

78:         uint256 amountToMint = amountToRecover * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-4"></a>[GAS-4] Use Custom Errors instead of Revert Strings to save Gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

Additionally, custom errors can be used inside and outside of contracts (including interfaces and libraries).

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:

> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Consider replacing **all revert strings** with custom errors in the solution, and particularly those that have multiple occurrences:

*Instances (7)*:
```solidity
File: contracts/asd/asdOFT.sol

45:         require(returnCode == 0, "Error when minting");

55:         require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying

69:             require(_amount <= maximumWithdrawable, "Too many tokens requested");

74:         require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdUSDC.sol

36:         require(whitelistedUSDCVersions[_usdcVersion], "ASDUSDC: USDC version not whitelisted");

54:         require(whitelistedUSDCVersions[_usdcVersion], "ASDUSDC: USDC version not whitelisted");

60:         require(usdcBalances[_usdcVersion] >= amountToWithdraw, "ASDUSDC: Not enough USDC balance");

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-5"></a>[GAS-5] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*Instances (1)*:
```solidity
File: contracts/asd/asdUSDC.sol

75:         uint amountToRecover = ERC20(_usdcVersion).balanceOf(address(this)) - usdcBalances[_usdcVersion];

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-6"></a>[GAS-6] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (3)*:
```solidity
File: contracts/asd/asdOFT.sol

63:     function withdrawCarry(uint256 _amount) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdUSDC.sol

24:     function updateWhitelist(address _usdcVersion, bool _isWhitelisted) external onlyOwner {

73:     function recover(address _usdcVersion) external onlyOwner returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-7"></a>[GAS-7] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (1)*:
```solidity
File: contracts/asd/asdRouter.sol

21:     uint public constant ambientPoolIdx = 36000;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="GAS-8"></a>[GAS-8] Use of `this` instead of marking as `public` an `external` function
Using `this.` is like making an expensive external call. Consider marking the called function as public

*Saves around 2000 gas per instance*

*Instances (3)*:
```solidity
File: contracts/asd/asdUSDC.sol

40:         uint256 amountToMint = _amount * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

58:         uint256 amountToWithdraw = _amount / (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

78:         uint256 amountToMint = amountToRecover * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="GAS-9"></a>[GAS-9] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (1)*:
```solidity
File: contracts/asd/asdRouter.sol

183:         if (_nativeAmount > 0 && _nativeAmount <= msg.value) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe | 3 |
| [NC-2](#NC-2) | Constants should be in CONSTANT_CASE | 1 |
| [NC-3](#NC-3) | `constant`s should be defined rather than using magic numbers | 4 |
| [NC-4](#NC-4) | Consider disabling `renounceOwnership()` | 2 |
| [NC-5](#NC-5) | Functions should not be longer than 50 lines | 23 |
| [NC-6](#NC-6) | Change int to int256 | 3 |
| [NC-7](#NC-7) | Change uint to uint256 | 14 |
| [NC-8](#NC-8) | Interfaces should be indicated with an `I` prefix in the contract name | 2 |
| [NC-9](#NC-9) | Lines are too long | 8 |
| [NC-10](#NC-10) | Consider using named mappings | 2 |
| [NC-11](#NC-11) | `address`s shouldn't be hard-coded | 1 |
| [NC-12](#NC-12) | Variable names that consist of all capital letters should be reserved for `constant`/`immutable` variables | 8 |
| [NC-13](#NC-13) | Use scientific notation for readability reasons for large multiples of ten | 1 |
| [NC-14](#NC-14) | Avoid the use of sensitive terms | 7 |
| [NC-15](#NC-15) | Use Underscores for Number Literals (add an underscore every 3 digits) | 4 |
### <a name="NC-1"></a>[NC-1] Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe
When using `abi.encodeWithSignature`, it is possible to include a typo for the correct function signature.
When using `abi.encodeWithSignature` or `abi.encodeWithSelector`, it is also possible to provide parameters that are not of the correct type for the function.

To avoid these pitfalls, it would be best to use [`abi.encodeCall`](https://solidity-by-example.org/abi-encode/) instead.

*Instances (3)*:
```solidity
File: contracts/asd/asdRouter.sol

157:             (bool successfulSend, bytes memory data) = payable(_payload._cantoAsdAddress).call{value: _payload._feeForSend}(abi.encodeWithSelector(IOFT.send.selector, sendParams, fee, _payload._cantoRefundAddress));

197:         (bool success, bytes memory errReason) = _asdVault.call(abi.encodeWithSelector(ASDOFT.mint.selector, _amountNote));

247:         (bool successSwap, bytes memory data) = crocSwapAddress.call(abi.encodeWithSelector(ICrocSwapDex.swap.selector, baseToken, quoteToken, ambientPoolIdx, !isNoteBase, !isNoteBase, amountConverted, 0, 0, uintMinAmount, 0));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="NC-2"></a>[NC-2] Constants should be in CONSTANT_CASE
For `constant` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (1)*:
```solidity
File: contracts/asd/asdRouter.sol

21:     uint public constant ambientPoolIdx = 36000;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="NC-3"></a>[NC-3] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*Instances (4)*:
```solidity
File: contracts/asd/asdOFT.sol

28:         if (block.chainid == 7700 || block.chainid == 7701) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

71:         if (composeMsg.length != 224) {

152:             bytes memory sendOptions = OptionsBuilder.addExecutorLzReceiveOption(OptionsBuilder.newOptions(), 200000, 0);

259:         uint POOL_PARAM_SLOT = 65545;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="NC-4"></a>[NC-4] Consider disabling `renounceOwnership()`
If the plan for your project does not include eventually giving up all ownership control, consider overwriting OpenZeppelin's `Ownable`'s `renounceOwnership()` function in order to disable it.

*Instances (2)*:
```solidity
File: contracts/asd/asdRouter.sol

17: contract ASDRouter is IOAppComposer, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

11: contract ASDUSDC is ERC20, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="NC-5"></a>[NC-5] Functions should not be longer than 50 lines
Overly complex code can make understanding functionality more difficult, try to further modularize your code to ensure readability 

*Instances (23)*:
```solidity
File: contracts/ambient/CrocInterfaces.sol

2:     function swap(address base, address quote, uint256 poolIdx, bool isBuy, bool inBaseQty, uint128 qty, uint16 tip, uint128 limitPrice, uint128 minOut, uint8 reserveFlags) external payable returns (int128 baseQuote, int128 quoteFlow);

4:     function readSlot(uint256 slot) external view returns (uint256 data);

8:     function calcImpact(address base, address quote, uint256 poolIdx, bool isBuy, bool inBaseQty, uint128 qty, uint16 poolTip, uint128 limitPrice) external view returns (int128 baseFlow, int128 quoteFlow, uint128 finalPrice);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/ambient/CrocInterfaces.sol)

```solidity
File: contracts/asd/asdOFT.sol

63:     function withdrawCarry(uint256 _amount) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

61:     function lzCompose(address _from, bytes32 _guid, bytes calldata _message, address _executor, bytes calldata _extraData) external payable {

122:     function _decodeOFTComposeMsg(bytes calldata _message) internal pure returns (uint64 _nonce, uint32 _srcEid, uint256 _amountLD, bytes32 _composeFrom, bytes memory _composeMsg) {

135:     function _sendASD(bytes32 _guid, OftComposeMessage memory _payload, uint _amount) internal {

177:     function _refundToken(bytes32 _guid, address _tokenAddress, address _refundAddress, uint _amount, uint _nativeAmount, string memory _reason) internal {

193:     function _depositNoteToASDVault(address _asdVault, uint _amountNote) internal returns (bool, string memory) {

209:     function _swapOFTForNote(address _oftAddress, uint _amount, uint _minAmountNote) internal returns (uint, bool) {

257:     function ambientPoolFor(address _baseToken, address _quoteToken, uint256 _poolIdx) internal view returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

24:     function updateWhitelist(address _usdcVersion, bool _isWhitelisted) external onlyOwner {

34:     function deposit(address _usdcVersion, uint256 _amount) external returns (uint256) {

52:     function withdraw(address _usdcVersion, uint256 _amount) external returns (uint256) {

73:     function recover(address _usdcVersion) external onlyOwner returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

```solidity
File: contracts/clm/CTokenInterfaces.sol

2:     function mint(uint256 mintAmount) external returns (uint256);

4:     function redeem(uint256 redeemTokens) external returns (uint256);

6:     function redeemUnderlying(uint256 redeemAmount) external returns (uint256);

8:     function exchangeRateStored() external view returns (uint256);

10:     function balanceOfUnderlying(address _owner) external view returns (uint256);

14:     function underlying() external view returns (address);

16:     function mint(uint256 mintAmount) external returns (uint256);

18:     function redeemUnderlying(uint256 redeemAmount) external returns (uint256);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/clm/CTokenInterfaces.sol)

### <a name="NC-6"></a>[NC-6] Change int to int256
Throughout the code base, some variables are declared as `int`. To favor explicitness, consider changing all instances of `int` to `int256`

*Instances (3)*:
```solidity
File: contracts/asd/asdRouter.sol

234:         int minAmountInt = int(_minAmountNote); // stack too deep fix

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

40:         uint256 amountToMint = _amount * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

78:         uint256 amountToMint = amountToRecover * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="NC-7"></a>[NC-7] Change uint to uint256
Throughout the code base, some variables are declared as `uint`. To favor explicitness, consider changing all instances of `uint` to `uint256`

*Instances (14)*:
```solidity
File: contracts/asd/asdRouter.sol

21:     uint public constant ambientPoolIdx = 36000;

38:     event LZReceived(bytes32 indexed _guid, address _from, bytes _message, address _executor, bytes _extraData, uint _value);

40:     event TokenRefund(bytes32 indexed _guid, address _tokenAddress, address _refundAddress, uint _amount, uint _nativeAmount, string _reason);

42:     event ASDSent(bytes32 indexed _guid, address _to, address _asdAddress, uint _amount, uint32 _dstEid, bool _lzSend);

87:         uint amountUSDC = ASDUSDC(asdUSDC).deposit(_from, amountLD);

90:         (uint amountNote, bool successfulSwap) = _swapOFTForNote(asdUSDC, amountUSDC, payload._minAmountASD);

135:     function _sendASD(bytes32 _guid, OftComposeMessage memory _payload, uint _amount) internal {

177:     function _refundToken(bytes32 _guid, address _tokenAddress, address _refundAddress, uint _amount, uint _nativeAmount, string memory _reason) internal {

193:     function _depositNoteToASDVault(address _asdVault, uint _amountNote) internal returns (bool, string memory) {

209:     function _swapOFTForNote(address _oftAddress, uint _amount, uint _minAmountNote) internal returns (uint, bool) {

259:         uint POOL_PARAM_SLOT = 65545;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

15:     event Deposit(address _version, uint _amount);

16:     event Withdrawal(address _version, uint _amount);

75:         uint amountToRecover = ERC20(_usdcVersion).balanceOf(address(this)) - usdcBalances[_usdcVersion];

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="NC-8"></a>[NC-8] Interfaces should be indicated with an `I` prefix in the contract name

*Instances (2)*:
```solidity
File: contracts/clm/CTokenInterfaces.sol

1: interface CTokenInterface {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/clm/CTokenInterfaces.sol)

```solidity
File: contracts/clm/Turnstile.sol

1: interface Turnstile {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/clm/Turnstile.sol)

### <a name="NC-9"></a>[NC-9] Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

*Instances (8)*:
```solidity
File: contracts/ambient/CrocInterfaces.sol

2:     function swap(address base, address quote, uint256 poolIdx, bool isBuy, bool inBaseQty, uint128 qty, uint16 tip, uint128 limitPrice, uint128 minOut, uint8 reserveFlags) external payable returns (int128 baseQuote, int128 quoteFlow);

8:     function calcImpact(address base, address quote, uint256 poolIdx, bool isBuy, bool inBaseQty, uint128 qty, uint16 poolTip, uint128 limitPrice) external view returns (int128 baseFlow, int128 quoteFlow, uint128 finalPrice);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/ambient/CrocInterfaces.sol)

```solidity
File: contracts/asd/asdOFT.sol

26:     constructor(string memory _name, string memory _symbol, address _lzEndpoint, address _cNote, address _csrRecipient) OFT(_name, _symbol, _lzEndpoint, msg.sender) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

122:     function _decodeOFTComposeMsg(bytes calldata _message) internal pure returns (uint64 _nonce, uint32 _srcEid, uint256 _amountLD, bytes32 _composeFrom, bytes memory _composeMsg) {

153:             SendParam memory sendParams = SendParam({dstEid: _payload._dstLzEid, to: OFTComposeMsgCodec.addressToBytes32(_payload._dstReceiver), amountLD: _amount, minAmountLD: _amount, extraOptions: sendOptions, composeMsg: "0x", oftCmd: "0x"});

157:             (bool successfulSend, bytes memory data) = payable(_payload._cantoAsdAddress).call{value: _payload._feeForSend}(abi.encodeWithSelector(IOFT.send.selector, sendParams, fee, _payload._cantoRefundAddress));

230:         (int128 baseFlow, int128 quoteFlow, ) = ICrocImpact(crocImpactAddress).calcImpact(baseToken, quoteToken, ambientPoolIdx, !isNoteBase, !isNoteBase, amountConverted, 0, 0);

247:         (bool successSwap, bytes memory data) = crocSwapAddress.call(abi.encodeWithSelector(ICrocSwapDex.swap.selector, baseToken, quoteToken, ambientPoolIdx, !isNoteBase, !isNoteBase, amountConverted, 0, 0, uintMinAmount, 0));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="NC-10"></a>[NC-10] Consider using named mappings
Consider moving to solidity version 0.8.18 or later, and using [named mappings](https://ethereum.stackexchange.com/questions/51629/how-to-name-the-arguments-in-mapping/145555#145555) to make it easier to understand the purpose of each mapping

*Instances (2)*:
```solidity
File: contracts/asd/asdUSDC.sol

12:     mapping(address => bool) public whitelistedUSDCVersions;

13:     mapping(address => uint256) public usdcBalances;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="NC-11"></a>[NC-11] `address`s shouldn't be hard-coded
It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

*Instances (1)*:
```solidity
File: contracts/asd/asdOFT.sol

30:             Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

### <a name="NC-12"></a>[NC-12] Variable names that consist of all capital letters should be reserved for `constant`/`immutable` variables
If the variable needs to be different based on which class it comes from, a `view`/`pure` *function* should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).

*Instances (8)*:
```solidity
File: contracts/ambient/CrocInterfaces.sol

1: interface ICrocSwapDex {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/ambient/CrocInterfaces.sol)

```solidity
File: contracts/asd/OFT.sol

1: pragma solidity ^0.8.22;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/OFT.sol)

```solidity
File: contracts/asd/OFTAdapter.sol

1: pragma solidity ^0.8.22;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/OFTAdapter.sol)

```solidity
File: contracts/asd/asdOFT.sol

1: pragma solidity ^0.8.22;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

1: pragma solidity ^0.8.22;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

1: pragma solidity ^0.8.22;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

```solidity
File: contracts/clm/CTokenInterfaces.sol

1: interface CTokenInterface {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/clm/CTokenInterfaces.sol)

```solidity
File: contracts/clm/Turnstile.sol

1: interface Turnstile {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/clm/Turnstile.sol)

### <a name="NC-13"></a>[NC-13] Use scientific notation for readability reasons for large multiples of ten
The more a number has zeros, the harder it becomes to see with the eyes if it's the intended value. To ease auditing and bug bounty hunting, consider using the scientific notation

*Instances (1)*:
```solidity
File: contracts/asd/asdRouter.sol

152:             bytes memory sendOptions = OptionsBuilder.addExecutorLzReceiveOption(OptionsBuilder.newOptions(), 200000, 0);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="NC-14"></a>[NC-14] Avoid the use of sensitive terms
Use [alternative variants](https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/), e.g. allowlist/denylist instead of whitelist/blacklist

*Instances (7)*:
```solidity
File: contracts/asd/asdRouter.sol

79:         if (!ASDUSDC(asdUSDC).whitelistedUSDCVersions(_from)) {

81:             _refundToken(_guid, _from, payload._cantoRefundAddress, amountLD, msg.value, "not whitelisted");

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

12:     mapping(address => bool) public whitelistedUSDCVersions;

24:     function updateWhitelist(address _usdcVersion, bool _isWhitelisted) external onlyOwner {

25:         whitelistedUSDCVersions[_usdcVersion] = _isWhitelisted;

36:         require(whitelistedUSDCVersions[_usdcVersion], "ASDUSDC: USDC version not whitelisted");

54:         require(whitelistedUSDCVersions[_usdcVersion], "ASDUSDC: USDC version not whitelisted");

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="NC-15"></a>[NC-15] Use Underscores for Number Literals (add an underscore every 3 digits)

*Instances (4)*:
```solidity
File: contracts/asd/asdOFT.sol

28:         if (block.chainid == 7700 || block.chainid == 7701) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

21:     uint public constant ambientPoolIdx = 36000;

152:             bytes memory sendOptions = OptionsBuilder.addExecutorLzReceiveOption(OptionsBuilder.newOptions(), 200000, 0);

259:         uint POOL_PARAM_SLOT = 65545;

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | `approve()`/`safeApprove()` may revert if the current approval is not zero | 3 |
| [L-2](#L-2) | Use a 2-step ownership transfer pattern | 2 |
| [L-3](#L-3) | `decimals()` is not a part of the ERC-20 standard | 3 |
| [L-4](#L-4) | Division by zero not prevented | 1 |
| [L-5](#L-5) | External call recipient may consume all transaction gas | 3 |
| [L-6](#L-6) | Loss of precision | 1 |
| [L-7](#L-7) | Unsafe ERC20 operation(s) | 6 |
| [L-8](#L-8) | Unsafe solidity low-level call can cause gas grief attack | 2 |
### <a name="L-1"></a>[L-1] `approve()`/`safeApprove()` may revert if the current approval is not zero
- Some tokens (like the *very popular* USDT) do not work when changing the allowance from an existing non-zero allowance value (it will revert if the current approval is not zero to protect against front-running changes of approvals). These tokens must first be approved for zero and then the actual allowance can be approved.
- Furthermore, OZ's implementation of safeApprove would throw an error if an approve is attempted from a non-zero value (`"SafeERC20: approve from non-zero to non-zero allowance"`)

Set the allowance to zero immediately before each of the existing allowance calls

*Instances (3)*:
```solidity
File: contracts/asd/asdOFT.sol

42:         note.approve(cNote, _amount);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

86:         IERC20(_from).approve(asdUSDC, amountLD);

195:         IERC20(noteAddress).approve(_asdVault, _amountNote);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="L-2"></a>[L-2] Use a 2-step ownership transfer pattern
Recommend considering implementing a two step process where the owner or admin nominates an account and the nominated account needs to call an `acceptOwnership()` function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account. Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

*Instances (2)*:
```solidity
File: contracts/asd/asdRouter.sol

17: contract ASDRouter is IOAppComposer, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

11: contract ASDUSDC is ERC20, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="L-3"></a>[L-3] `decimals()` is not a part of the ERC-20 standard
The `decimals()` function is not a part of the [ERC-20 standard](https://eips.ethereum.org/EIPS/eip-20), and was added later as an [optional extension](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/IERC20Metadata.sol). As such, some valid ERC20 tokens do not support this interface, so it is unsafe to blindly cast all tokens to this interface, and then call this function.

*Instances (3)*:
```solidity
File: contracts/asd/asdUSDC.sol

40:         uint256 amountToMint = _amount * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

58:         uint256 amountToWithdraw = _amount / (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

78:         uint256 amountToMint = amountToRecover * (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="L-4"></a>[L-4] Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

*Instances (1)*:
```solidity
File: contracts/asd/asdUSDC.sol

58:         uint256 amountToWithdraw = _amount / (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="L-5"></a>[L-5] External call recipient may consume all transaction gas
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

*Instances (3)*:
```solidity
File: contracts/asd/asdRouter.sol

157:             (bool successfulSend, bytes memory data) = payable(_payload._cantoAsdAddress).call{value: _payload._feeForSend}(abi.encodeWithSelector(IOFT.send.selector, sendParams, fee, _payload._cantoRefundAddress));

197:         (bool success, bytes memory errReason) = _asdVault.call(abi.encodeWithSelector(ASDOFT.mint.selector, _amountNote));

247:         (bool successSwap, bytes memory data) = crocSwapAddress.call(abi.encodeWithSelector(ICrocSwapDex.swap.selector, baseToken, quoteToken, ambientPoolIdx, !isNoteBase, !isNoteBase, amountConverted, 0, 0, uintMinAmount, 0));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="L-6"></a>[L-6] Loss of precision
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

*Instances (1)*:
```solidity
File: contracts/asd/asdUSDC.sol

58:         uint256 amountToWithdraw = _amount / (10 ** (this.decimals() - ERC20(_usdcVersion).decimals()));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

### <a name="L-7"></a>[L-7] Unsafe ERC20 operation(s)

*Instances (6)*:
```solidity
File: contracts/asd/asdOFT.sol

42:         note.approve(cNote, _amount);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

86:         IERC20(_from).approve(asdUSDC, amountLD);

140:             ASDOFT(_payload._cantoAsdAddress).transfer(_payload._dstReceiver, _amount);

181:         IERC20(_tokenAddress).transfer(_refundAddress, _amount);

184:             payable(_refundAddress).transfer(_nativeAmount);

195:         IERC20(noteAddress).approve(_asdVault, _amountNote);

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

### <a name="L-8"></a>[L-8] Unsafe solidity low-level call can cause gas grief attack
Using the low-level calls of a solidity address can leave the contract open to gas grief attacks. These attacks occur when the called contract returns a large amount of data.

So when calling an external contract, it is necessary to check the length of the return data before reading/copying it (using `returndatasize()`).

*Instances (2)*:
```solidity
File: contracts/asd/asdRouter.sol

197:         (bool success, bytes memory errReason) = _asdVault.call(abi.encodeWithSelector(ASDOFT.mint.selector, _amountNote));

247:         (bool successSwap, bytes memory data) = crocSwapAddress.call(abi.encodeWithSelector(ICrocSwapDex.swap.selector, baseToken, quoteToken, ambientPoolIdx, !isNoteBase, !isNoteBase, amountConverted, 0, 0, uintMinAmount, 0));

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 5 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (5)*:
```solidity
File: contracts/asd/asdOFT.sol

63:     function withdrawCarry(uint256 _amount) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdOFT.sol)

```solidity
File: contracts/asd/asdRouter.sol

17: contract ASDRouter is IOAppComposer, Ownable {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdRouter.sol)

```solidity
File: contracts/asd/asdUSDC.sol

11: contract ASDUSDC is ERC20, Ownable {

24:     function updateWhitelist(address _usdcVersion, bool _isWhitelisted) external onlyOwner {

73:     function recover(address _usdcVersion) external onlyOwner returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-03-canto/blob/main/contracts/asd/asdUSDC.sol)

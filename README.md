This article is going to give you a couple of tips to reduce gas consumption in your smart contracts.

It’s very important to understand how to do this because if it costs too much money to execute a function in your smart contract, fewer users will be willing to run your Dapp.

## 1: Use `immutable` and `constant`
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving `SLOAD`.
```solidity
contract MyContract {
    uint256 constant y = 10;
    uint256 immutable x;
    constructor() {
        x = 5;
    } 
 }
```
<br />

## 2: Use calldata instead of memory for function parameters 

It is generally cheaper to load variables directly from calldata, rather than copying them to memory. Only use memory if the variable needs to be modified.
Before:
```solidity
function loop(uint[] memory arr) external pure returns (uint sum) {
    for (uint i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
}
```

After:
```solidity
function loop(uint[] calldata arr) external pure returns (uint sum) {
    for (uint i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
}
```
<br />

## 3: Use custom errors to save deployment and runtime costs in case of revert.

Instead of using strings for error messages `(e.g., require(msg.sender == owner, “unauthorized”))`, you can use custom errors to reduce both deployment and runtime gas costs. In addition, they are very convenient as you can easily pass dynamic information to them.
Before:
```solidity
function add(uint256 _amount) public {
    require(msg.sender == owner, "unauthorized");
- This is how mine looked like.
    total += _amount;
}
```
After:
```solidity
error Unauthorized(address caller);
function add(uint256 _amount) public {
    if (msg.sender != owner)
        revert Unauthorized(msg.sender);
    total += _amount;
}
```
<br />

## 4: Short-circuit with || and &&
For || and && operators, the second case is not checked if the first case gives the result of the logical expression.  Put the lower-cost expression first so the higher-cost expression may be skipped (short-circuit).

Before:
```solidity
case 1: 100 gas
case 2: 50 gas
if(case1 && case2) revert 
if(case1 || case2) revert
```
After:
```solidity
if(case2 && case1) revert 
if(case2 || case1) revert 
```

## 5:Use of constant keccak variables results in extra hashing (and so gas).
This results in the keccak operation being performed whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to immutable will only perform hashing on contract deployment which will save gas.

### reference

[ethereum/solidity#9232 (comment)](https://github.com/ethereum/solidity/issues/9232#issuecomment-646131646)

```diff
- bytes32 public constant SET_MANAGER_WITHDRAW_HOOK_ROLE = keccak256("Collateral_setManagerWithdrawHook(ICollateralHook)");
+ bytes32 public immutable SET_MANAGER_WITHDRAW_HOOK_ROLE = keccak256("Collateral_setManagerWithdrawHook(ICollateralHook)");
```

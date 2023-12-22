### 1. Don't calculate constants
```solidity
uint256 constant INT_MAX = uint256(type(int256).max);
int256 constant MIN_BALANCE = 10**12;
int256 constant MULTIPLIER = 1e18;
```

### 2. Do not initialize variables with default values
```solidity
bool constant FEE_DOWN = false;
Address SampleAddress = 0;
```

### 3. OR in `if-`condition can be rewritten to two single if conditions
```solidity
if (py_init >= MAX_PRICE_VALUE || py_final >= MAX_PRICE_VALUE) return 1;
if (px_init <= MIN_PRICE_VALUE || px_final <= MIN_PRICE_VALUE) return 1;
```
This suggests, that conditions should be rewritten to:
```solidity
if (py_init >= MAX_PRICE_VALUE) return 1;
if (py_final >= MAX_PRICE_VALUE) return 1;
if (px_init <= MIN_PRICE_VALUE) return 1;
if (px_final <= MIN_PRICE_VALUE) return 1;
```

### 4. Use `!= 0` instead of `> 0` for unsigned integer comparison
When dealing with unsigned integer types, comparisons with `!= 0` are cheaper then with `> 0`.

![](https://global.discourse-cdn.com/business6/uploads/zeppelin/original/2X/3/363a367d6d68851f27d2679d10706cd16d788b96.png)

> [!IMPORTANT]
> To update this with using at least 0.8.6 there is no difference in gas usage with `!= 0` or `> 0`

### 5. Use shift right/left instead of division/multiplication if possible
While the `DIV` / `MUL` opcode uses 5 gas, the `SHR` / `SHL` opcode only uses 3 gas. Furthermore, beware that Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting. Eventually, overflow checks are never performed for shift operations as they are done for arithmetic operations. Instead, the result is always truncated.
![](https://global.discourse-cdn.com/business6/uploads/zeppelin/original/2X/c/c427c82eac6126e75c9f4092a0354b0fa56840a2.jpeg)

### 6. Caching the length in for loops
Consider a generic example of an array `arr` and the following loop:
```solidity
for (uint i = 0; i < arr.length; i++) { 
    // do something that doesn't change arr.length
}
```
In the above case, the solidity compiler will always read the length of the array during each iteration. That is,

* if it is a `storage` array, this is an extra `sload` operation (100 additional extra gas ([EIP-2929](https://eips.ethereum.org/EIPS/eip-2929) 17) for each iteration except for the first),
* if it is a `memory` array, this is an extra `mload` operation (3 additional gas for each iteration except for the first),
* if it is a `calldata` array, this is an extra `calldataload` operation (3 additional gas for each iteration except for the first)

This extra costs can be avoided by caching the array length (in stack):
```solidity
uint length = arr.length;
for (uint i = 0; i < length; i++) {
    // do something that doesn't change arr.length
}
```
In the above example, the `sload` or `mload` or `calldataload` operation is only called once and subsequently replaced by a cheap `dupN` instruction. Even though mload , `calldataload` and `dupN` have the same gas cost, `mload` and `calldataload` needs an additional `dupN` to put the offset in the stack, i.e., an extra 3 gas.

This optimization is especially important if it is a storage array or if it is a lengthy for loop.

Note that the Yul based optimizer (not enabled by default; only relevant if you are using `--experimental-via-ir` or the equivalent in standard JSON) can sometimes do this caching automatically. However, this is likely not the case in your project. [Reference](https://forum.soliditylang.org/t/solidity-team-ama-2-on-wed-10th-of-march-2021/152/15?u=hrkrshnn). Also see [this](https://gist.github.com/hrkrshnn/a1165fc31cbbf1fae9f271c73830fdda).

### 7. Use `calldata` instead of `memory` for function parameters
In some cases, having function arguments in `calldata` instead of `memory` is more optimal.

Consider the following generic example:
```solidity
contract C {
    function add(uint[] memory arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```
In the above example, the dynamic array `arr` has the storage location `memory` . When the function gets called externally, the array values are kept in `calldata` and copied to `memory` during ABI decoding (using the opcode `calldataload` and mstore ). And during the for loop, `arr[i]` accesses the value in memory using a `mload` . However, for the above example, this is inefficient. Consider the following snippet instead:
```solidity
contract C {
    function add(uint[] calldata arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```
In the above snippet, instead of going via memory, the value is directly read from `calldata` using `calldataload` . That is, there are no intermediate memory operations that carry this value.

**Gas savings** : In the former example, the ABI decoding begins with copying value from `calldata` to `memory` in a for loop. Each iteration would cost at least 60 gas. In the latter example, this can be completely avoided. This will also reduce the number of instructions and therefore reduce the deployment time cost of the contract.

In short, use `calldata` instead of `memory` if the function argument is only read.

Note that in older Solidity versions, changing some function arguments from `memory` to `calldata` may cause _**unimplemented feature error**_. This can be avoided by using a newer ( `0.8.*` ) Solidity compiler.

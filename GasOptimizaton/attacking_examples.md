## DoS(Denial of Services)

### Gas Limitation Attacking

https://medium.com/valixconsulting/solidity-security-by-example-10-denial-of-service-with-gas-limit-346e87e2ef78


### Gas Limit DoS via Block Stuffing
https://consensys.github.io/smart-contract-best-practices/attacks/denial-of-service/#:~:text=if%20absolutely%20necessary.-,Gas%20Limit%20DoS%20on%20the%20Network%20via%20Block%20Stuffing,a%20high%20enough%20gas%20price.

Even if your contract does not contain an unbounded loop, an attacker can prevent other transactions from being included in the blockchain for several blocks by placing computationally intensive transactions with a high enough gas price.

A Block Stuffing attack can be used on any contract requiring an action within a certain time period. However, as with any attack, it is only profitable when the expected reward exceeds its cost. The cost of this attack is directly proportional to the number of blocks which need to be stuffed. If a large payout can be obtained by preventing actions from other participants, your contract will likely be targeted by such an attack.

> [!IMPORTANT]
> Stuffing a whole block with dummy transactions is very cheap on Binance Smart Chain.
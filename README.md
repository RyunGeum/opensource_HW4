# opensource-HW4 오프소스 개발프로젝트 과제#4

## 1. 개발 내용


### ● HTLC lock 소스코드를 활용하여 ethereum testnet에서 ethereum wallet 계정에서 스마트 컨트랙트로 동작시키고, 이를 Web3.js 로 연계 호출하여 동작하는 애플리케이션을 개발.


1-1. ethereum 과 연동된 metamask wallet 계정을 만든다.

![image](https://github.com/2022039078/opensource-HW4/assets/131639123/67c6dea9-bbeb-445e-9ff0-94a8a29ecaf3)


1-2. HTCL lock 소스 코드를 활용해 ethereum wallet 계정에서 스마트 컨트랙트로 동작시킨다.

1-3. Web3.js로 연계 호출하여 동작한다.


## 2. 동작 원리


### ● 개발 환경: Node.js, HTLC.sol 이용

2-1. Node.js
```
// main.js
require(['module1', 'module2'], function(module1, module2) {
    const Web3 = require('web3');
const fs = require('fs');

// Connect to Ethereum testnet using Infura
const web3 = new Web3(new Web3.providers.HttpProvider('https://rinkeby.infura.io/v3/YOUR_INFURA_API_KEY'));

// Load compiled contract ABI and bytecode
const contractABI = JSON.parse(fs.readFileSync('path/to/HTLC_sol_HTLC.abi', 'utf-8'));
const contractBytecode = fs.readFileSync('path/to/HTLC_sol_HTLC.bin', 'utf-8');

// Ethereum wallet private key (for sender)
const privateKey = ' ';

// Sender's Ethereum wallet address
const senderAddress = 'SENDER_WALLET_ADDRESS';

// Receiver's Ethereum wallet address
const receiverAddress = 'RECEIVER_WALLET_ADDRESS';

// Hash of the secret preimage
const hashLock = '0xYOUR_HASH_LOCK';

// Duration for which funds are locked (in seconds)
const duration = 86400; // 1 day

// Create contract instance
const contract = new web3.eth.Contract(contractABI);

// Deploy the contract
const deployContract = async () => {
    const senderAccount = web3.eth.accounts.privateKeyToAccount( );
    const gas = await contract.deploy({
        data: '0x' + contractBytecode,
        arguments: [receiverAddress, hashLock, duration],
    }).estimateGas();

    const deployedContract = await contract.deploy({
        data: '0x' + contractBytecode,
        arguments: [receiverAddress, hashLock, duration],
    }).send({
        from: senderAccount.address,
        gas: gas,
    });

    console.log('Contract deployed at:', deployedContract.options.address);
};

// Claim funds by providing the preimage
const claimFunds = async (preimage) => {
    const receiverAccount = web3.eth.accounts.privateKeyToAccount( );
    const instance = new web3.eth.Contract(contractABI, 'CONTRACT_ADDRESS');
    const gas = await instance.methods.claimFunds(preimage).estimateGas();

    await instance.methods.claimFunds(preimage).send({
        from: receiverAccount.address,
        gas: gas,
    });

    console.log('Funds claimed successfully.');
};

// Refund funds to the sender
const refundFunds = async () => {
    const senderAccount = web3.eth.accounts.privateKeyToAccount( );
    const instance = new web3.eth.Contract(contractABI, 'CONTRACT_ADDRESS');
    const gas = await instance.methods.refund().estimateGas();

    await instance.methods.refund().send({
        from: senderAccount.address,
        gas: gas,
    });

    console.log('Funds refunded successfully.');
};

// Deploy the contract and interact with it
deployContract()
    .then(() => claimFunds('PREIMAGE_FOR_CLAIMING'))
    .catch((error) => console.error('Error:', error));

  });
```

2-2. HTLC.sol
```
// HTLC.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HTLC {
    address public sender;
    address public receiver;
    bytes32 public hashLock;
    uint public expiration;

    event FundsClaimed(address indexed _claimer, uint _amount);

    constructor(address _receiver, bytes32 _hashLock, uint _duration) {
        sender = msg.sender;
        receiver = _receiver;
        hashLock = _hashLock;
        expiration = block.timestamp + _duration;
    }

    modifier onlySender() {
        require(msg.sender == sender, "Only sender can call this");
        _;
    }

    modifier onlyReceiver() {
        require(msg.sender == receiver, "Only receiver can call this");
        _;
    }

    modifier onlyWithHash(bytes32 _preimage) {
        require(keccak256(abi.encodePacked(_preimage)) == hashLock, "Invalid preimage");
        _;
    }

    function claimFunds(bytes32 _preimage) external onlyReceiver onlyWithHash(_preimage) {
        require(block.timestamp <= expiration, "Funds expired");
        emit FundsClaimed(receiver, address(this).balance);

        // Transfer funds to the receiver
        payable(receiver).transfer(address(this).balance);
    }

    function refund() external onlySender {
        require(block.timestamp > expiration, "Funds still locked");

        // Transfer funds back to the sender
        payable(sender).transfer(address(this).balance);
    }
}
```

## 3. 컴파일 및 실행 방법

3-1. HTLC 스마트 컨트랙트를 Solidity 언어로 작성한다.
Remix IDE (https://remix.ethereum.org/) 를 사용했다.
Remix IDE에서 HTLC.sol 파일을 생성하고 컴파일한다.

3-2. Web3.js를 사용하여 이 스마트 컨트랙트와 상호작용하는 JavaScript 코드를 작성한다.

3-3. Infura를 사용하여 Ethereum testnet에 연결하고, 프라이빗 키, 주소, 해시락 등을 설정한다. 

3-4. 컨트랙트를 배포하고, 성공적으로 배포된 경우 자금을 청구하거나 환불한다.

3-5. 주의: 실제 프로덕션에서는 보안을 유지하기 위해 프라이빗 키를 매우 안전하게 다루어야 한다.

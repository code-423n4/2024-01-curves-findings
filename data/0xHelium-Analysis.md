The audit encompass the following files, each integral to the functioning of the protocol:

Curves.sol: This is the primary file of the Curves protocol. It contains the core logic and functions that define the overall behavior and rules of the system. This file started as a fork of friend.tech FriendtechSharesV1.sol

CurvesERC20.sol: This file defines the ERC20 token that will be created upon exporting a Curve token. It outlines the token's properties and behaviors consistent with the ERC20 standard.

CurvesERC20Factory.sol: Serving as an ERC20 factory, this file abstracts the logic for ERC20 token creation. Its primary purpose is to streamline the token creation process and reduce the overall footprint of the protocol, ensuring efficiency and scalability.

FeeSplitter.sol: This script manages the distribution of fees. It is responsible for the fair and accurate division of transaction fees amongst token holders, in line with the protocol's incentive structure.

Security.sol: This file standardizes the security criteria for the protocol. It includes protocols and measures designed to safeguard the system against vulnerabilities and ensure compliance with established security standards.

![Imgur](https://i.imgur.com/JGejOXl.png)

The Curves protocol, is an extension of friend.tech, introduces several innovative features.
It works with a set of 5 smart contracts that interact between them like this

![Imgur](https://i.imgur.com/YNsNQGN.png)

For context on friend.tech let's see the following
## What is Friend.tech?

Friend.tech is changing the way we think about social networking. Instead of just connecting friends, it does something different. It turns your connections into something valuable called “Shares” or “Keys.” Imagine being able to invest in your friend’s social network or in a group of users to show trustworthiness. It’s a different kind of social app.

In this system, every user on X can becomes a social token. You can purchase these tokens, sell them for a profit, or keep them as their worth increases with the reputation of the users they represent.

But that’s not all—Shares provide access to special chat rooms for influencers, premium content, and more.

Friend.tech belongs to the Socialfi sector of blockchain applications, making it a truly unique and exciting addition to the decentralized world.
## Friend.tech Smart Contract Breakdown
The FriendtechSharesV1 contract is like a compact powerhouse, with only about 90 lines of code (excluding the Ownable contract). This small contract contains five crucial variables, one event, and ten functions. What’s even more impressive is that this concise codebase can handle substantial assets.

Below, we provide the FriendtechSharesV1 contract code for your examination. Looking into the code will provide you with a better understanding of how it enables the innovative features of the Friend.tech platform.
```
pragma solidity >=0.8.2 <0.9.0;

// TODO: Events, final pricing model, 

contract FriendtechSharesV1 is Ownable {
    address public protocolFeeDestination;
    uint256 public protocolFeePercent;
    uint256 public subjectFeePercent;

    event Trade(address trader, address subject, bool isBuy, uint256 shareAmount, uint256 ethAmount, uint256 protocolEthAmount, uint256 subjectEthAmount, uint256 supply);

    // SharesSubject => (Holder => Balance)
    mapping(address => mapping(address => uint256)) public sharesBalance;

    // SharesSubject => Supply
    mapping(address => uint256) public sharesSupply;

    function setFeeDestination(address _feeDestination) public onlyOwner {
        protocolFeeDestination = _feeDestination;
    }

    function setProtocolFeePercent(uint256 _feePercent) public onlyOwner {
        protocolFeePercent = _feePercent;
    }

    function setSubjectFeePercent(uint256 _feePercent) public onlyOwner {
        subjectFeePercent = _feePercent;
    }

    function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : (supply - 1 )* (supply) * (2 * (supply - 1) + 1) / 6;
        uint256 sum2 = supply == 0 && amount == 1 ? 0 : (supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1) / 6;
        uint256 summation = sum2 - sum1;
        return summation * 1 ether / 16000;
    }

    function getBuyPrice(address sharesSubject, uint256 amount) public view returns (uint256) {
        return getPrice(sharesSupply[sharesSubject], amount);
    }

    function getSellPrice(address sharesSubject, uint256 amount) public view returns (uint256) {
        return getPrice(sharesSupply[sharesSubject] - amount, amount);
    }

    function getBuyPriceAfterFee(address sharesSubject, uint256 amount) public view returns (uint256) {
        uint256 price = getBuyPrice(sharesSubject, amount);
        uint256 protocolFee = price * protocolFeePercent / 1 ether;
        uint256 subjectFee = price * subjectFeePercent / 1 ether;
        return price + protocolFee + subjectFee;
    }

    function getSellPriceAfterFee(address sharesSubject, uint256 amount) public view returns (uint256) {
        uint256 price = getSellPrice(sharesSubject, amount);
        uint256 protocolFee = price * protocolFeePercent / 1 ether;
        uint256 subjectFee = price * subjectFeePercent / 1 ether;
        return price - protocolFee - subjectFee;
    }

    function buyShares(address sharesSubject, uint256 amount) public payable {
        uint256 supply = sharesSupply[sharesSubject];
        require(supply > 0 || sharesSubject == msg.sender, "Only the shares' subject can buy the first share");
        uint256 price = getPrice(supply, amount);
        uint256 protocolFee = price * protocolFeePercent / 1 ether;
        uint256 subjectFee = price * subjectFeePercent / 1 ether;
        require(msg.value >= price + protocolFee + subjectFee, "Insufficient payment");
        sharesBalance[sharesSubject][msg.sender] = sharesBalance[sharesSubject][msg.sender] + amount;
        sharesSupply[sharesSubject] = supply + amount;
        emit Trade(msg.sender, sharesSubject, true, amount, price, protocolFee, subjectFee, supply + amount);
        (bool success1, ) = protocolFeeDestination.call{value: protocolFee}("");
        (bool success2, ) = sharesSubject.call{value: subjectFee}("");
        require(success1 && success2, "Unable to send funds");
    }
    function sellShares(address sharesSubject, uint256 amount) public payable {
        uint256 supply = sharesSupply[sharesSubject];
        require(supply > amount, "Cannot sell the last share");
        uint256 price = getPrice(supply - amount, amount);
        uint256 protocolFee = price * protocolFeePercent / 1 ether;
        uint256 subjectFee = price * subjectFeePercent / 1 ether;
        require(sharesBalance[sharesSubject][msg.sender] >= amount, "Insufficient shares");
        sharesBalance[sharesSubject][msg.sender] = sharesBalance[sharesSubject][msg.sender] - amount;
        sharesSupply[sharesSubject] = supply - amount;
        emit Trade(msg.sender, sharesSubject, false, amount, price, protocolFee, subjectFee, supply - amount);
        (bool success1, ) = msg.sender.call{value: price - protocolFee - subjectFee}("");
        (bool success2, ) = protocolFeeDestination.call{value: protocolFee}("");
        (bool success3, ) = sharesSubject.call{value: subjectFee}("");
        require(success1 && success2 && success3, "Unable to send funds");
    }
}
```
In the code snippet above, the FriendtechSharesV1 contract inherits from the Ownable contract, which is one of the popular libraries provided by OpenZeppelin. This inheritance equips the contract with functionality for ownership management, enabling the contract owner to execute specific privileged actions.

Furthermore, upon inspecting the code, it becomes evident that all variables and functions within the contract have their visibility set to “public”. This visibility setting allows both external and internal calls to access these variables and functions.

After inspecting the code above, let’s dive into more details about the key components of the FriendtechSharesV1 contract.

## Variables
```
contract FriendtechSharesV1 is Ownable {
    address public protocolFeeDestination;
    uint256 public protocolFeePercent;
    uint256 public subjectFeePercent;

    // (...SNIPPET...)

    // SharesSubject => (Holder => Balance)
    mapping(address => mapping(address => uint256)) public sharesBalance;

    // SharesSubject => Supply
    mapping(address => uint256) public sharesSupply;
    
    // (...SNIPPET...)
}
```
The FriendtechSharesV1 contract includes five variables (excluding the _owner variable inherited from the Ownable contract). Here is the list of variables:

protocolFeeDestination:
This address variable stores the destination address for receiving protocol fees.

protocolFeePercent:
This variable represents the percentage of protocol fees.

subjectFeePercent:
This variable represents the percentage of subject fees.

sharesBalance:
This mapping associates shares subjects (addresses) with their respective holders and their share balances.

sharesSupply:
This mapping tracks the supply of shares for each shares subject.

These variables, protocolFeeDestination, protocolFeePercent, subjectFeePercent, sharesBalance, and sharesSupply, have public visibility, making it easy to track or view their values externally.

## Events

The FriendtechSharesV1 contract provides a single event named Trade. This event is triggered whenever shares are traded using the buyShares and sellShares functions.

The Trade event logs various details about each trade, including:

The trader’s address
The subject’s address
The trade type (buy or sell)
The amounts of shares and ether involved
Details about protocol and subject fees
The share supply

## Functions
```
contract FriendtechSharesV1 is Ownable {
    address public protocolFeeDestination;
    uint256 public protocolFeePercent;
    uint256 public subjectFeePercent;
    
    // (...SNIPPET...)
    
    function setFeeDestination(address _feeDestination) public onlyOwner {
        protocolFeeDestination = _feeDestination;
    }

    function setProtocolFeePercent(uint256 _feePercent) public onlyOwner {
        protocolFeePercent = _feePercent;
    }

    function setSubjectFeePercent(uint256 _feePercent) public onlyOwner {
        subjectFeePercent = _feePercent;
    }
    
    // (...SNIPPET...)
```
In the code snippet above, we list the functions that are accessible exclusively to the contract owner. These functions grant the contract owner the ability to configure three crucial aspects related to the fees on this platform:

setFeeDestination:
This function enables the contract owner to designate the destination address for receiving protocol fees
setProtocolFeePercent:
This function grants the contract owner the authority to set the percentage of protocol fees
setSubjectFeePercent:
This function provides the contract owner with the capability to establish the percentage of subject fees
🚨Remarkably, these three functions do not include input validation, maximum fee constraints, or time-lock mechanisms. Consequently, the contract owner has the freedom to set these values according to their preferences.

```
contract FriendtechSharesV1 is Ownable {
    // (...SNIPPET...)
    
    function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {
        uint256 sum1 = supply == 0 ? 0 : (supply - 1 )* (supply) * (2 * (supply - 1) + 1) / 6;
        uint256 sum2 = supply == 0 && amount == 1 ? 0 : (supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1) / 6;
        uint256 summation = sum2 - sum1;
        return summation * 1 ether / 16000;
    }
    
    function getBuyPrice(address sharesSubject, uint256 amount) public view returns (uint256) {
        return getPrice(sharesSupply[sharesSubject], amount);
    }

    function getSellPrice(address sharesSubject, uint256 amount) public view returns (uint256) {
        return getPrice(sharesSupply[sharesSubject] - amount, amount);
    }
	
    // (...SNIPPET...)
}
```
Next, the functions above work together to determine the prices at which shares can be bought and sold within the FriendtechSharesV1 contract.

getPrice:
This is a pure function that calculates the price based on the supply and the amount of shares being bought or sold
getBuyPrice:
This is a view function that retrieves the buy price of shares for a specific shares subject. It utilizes the getPrice function with the current supply and the desired amount of shares to be purchased
getSellPrice:
This function is similar to the getBuyPrice function and is also a view function. It calculates the sell price of shares for a specific shares subject by using the getPrice function with the adjusted supply after selling the specified amount of shares
These functions work together to determine the prices at which shares can be bought and sold within the FriendtechSharesV1 contract.

The interesting function of this part is the getPrice function. This function determines share prices based on the current supply of shares and the number of shares being evaluated.

Let’s provide a specific example of how the getPrice function works with given values for supply and amount.

```
// Suppose we have the following values and calculate the price:
supply = 100
amount = 1

// sum1 is calculated for the initial supply of 100:
sum1 = (100 - 1) * 100 * (2 * (100 - 1) + 1) / 6 = 161,700

// sum2 is calculated when we consider an additional amount of 1 share:
sum2 = (100 - 1 + 1) * (100 + 1) * (2 * (100 - 1 + 1) + 1) / 6 = 161,701

// summation is the difference between sum2 and sum1:
summation = 161,701 - 161,700 = 1

// convert summation to wei using the factor 1 ether / 16000:
price = 1 * (1 ether / 16000) = 62,500,000,000,000,000 wei // 0.0625 ETH
```
So, with a supply of 100 and an additional amount of 1, the price of shares would be 62,500,000,000,000,000 wei or 0.0625 ETH.

Relationship between Price and Supply in Friend.tech

Effect Of Buying (Increasing Supply):
When the supply of shares (supply) increases, the price of each additional share (amount) also increases. This relationship is quadratic, meaning that as the supply grows, the price increase becomes steeper.
Effect Of Selling (Decreasing Supply):
If the supply decreases (for example, if shares are sold), the price of each additional share will decrease.

![Imgur](https://i.imgur.com/e82KHL1.png)

As in the relationship between price and supply above, the getPrice function explains how share prices shift when more people want to buy them or when more people want to sell them. When there's high demand with many buyers, prices rise, but they don't always rise at a consistent rate. Sometimes they go up quickly, and other times they rise more slowly. These price changes can cause shares to fluctuate in value, going both up and down.

Next, we have two convenient functions: getBuyPriceAfterFee and getSellPriceAfterFee. These functions are intended for external services or applications to calculate the buy and sell prices of shares after accounting for protocol and subject fees.

While they don't directly impact the contract's internal state, they provide valuable information for users or external services to understand the final prices they will pay or receive, considering all associated fees.

In the final section of the FriendtechSharesV1 contract, we encounter two crucial functions: buyShares and sellShares.

These functions are at the heart of the contract's functionality, allowing users to interact with shares on the platform by buying and selling them.

## The buyShares function:
```
contract FriendtechSharesV1 is Ownable {
    // (...SNIPPET...)
    
    function buyShares(address sharesSubject, uint256 amount) public payable {
        uint256 supply = sharesSupply[sharesSubject];
        require(supply > 0 || sharesSubject == msg.sender, "Only the shares' subject can buy the first share");
        uint256 price = getPrice(supply, amount);
        uint256 protocolFee = price * protocolFeePercent / 1 ether;
        uint256 subjectFee = price * subjectFeePercent / 1 ether;
        require(msg.value >= price + protocolFee + subjectFee, "Insufficient payment");
        sharesBalance[sharesSubject][msg.sender] = sharesBalance[sharesSubject][msg.sender] + amount;
        sharesSupply[sharesSubject] = supply + amount;
        emit Trade(msg.sender, sharesSubject, true, amount, price, protocolFee, subjectFee, supply + amount);
        (bool success1, ) = protocolFeeDestination.call{value: protocolFee}("");
        (bool success2, ) = sharesSubject.call{value: subjectFee}("");
        require(success1 && success2, "Unable to send funds");
    }
    
    // (...SNIPPET...)
}
```
The buyShares function allows users to purchase shares from a specified shares subject by sending a certain amount of Ether as payment. This function calculates the price of the shares, which includes protocol and subject fees, and ensures that the sender provides enough Ether to complete the purchase. If the payment is sufficient, it updates the user's shares balance and the total supply of shares for the subject, logs the trade details, and transfers fees to the protocol and subject.

Let’s break it down line by line:

Retrieve Supply:
uint256 supply = sharesSupply[sharesSubject];
This line retrieves the current supply of shares for the specified shares subject.
Check Validity:
require(supply > 0 || sharesSubject == msg.sender, "Only the shares' subject can buy the first share");
This line checks whether the supply is greater than 0 or if the sender is the shares subject. If the supply is 0 (no shares exist) and the sender is not the shares subject, it prevents anyone except the subject from buying the first share.
Calculate Price:
uint256 price = getPrice(supply, amount);
This line calculates the price of the shares based on the current supply and the desired amount of shares to purchase using the getPrice function.
Calculate Protocol Fee:
uint256 protocolFee = price * protocolFeePercent / 1 ether;
Here, it calculates the protocol fee based on the price and the protocol fee percentage.
Calculate Subject Fee:
uint256 subjectFee = price * subjectFeePercent / 1 ether;
This line calculates the subject fee based on the price and the subject fee percentage.
Check Payment:
require(msg.value >= price + protocolFee + subjectFee, "Insufficient payment");
This checks if the Ether sent by the sender (msg.value) is greater than or equal to the total price, including protocol and subject fees. If the payment is insufficient, the function will not proceed.
Update User Balance:
sharesBalance[sharesSubject][msg.sender] = sharesBalance[sharesSubject][msg.sender] + amount;
It updates the user's balance of shares within the specified shares subject.
Update Supply:
sharesSupply[sharesSubject] = supply + amount;
This line increases the total supply of shares for the shares subject by the purchased amount.
Emit Trade Event:
emit Trade(msg.sender, sharesSubject, true, amount, price, protocolFee, subjectFee, supply + amount);
It emits a trade event with details about the transaction, including the sender, shares subject, trade type (buy), the amount purchased, price, fees, and summation of supply.
Transfer Funds:
(bool success1, ) = protocolFeeDestination.call{value: protocolFee}("");
This line attempts to transfer the protocol fee to the designated fee destination address.
Transfer Funds:
(bool success2, ) = sharesSubject.call{value: subjectFee}("");
Here, it attempts to transfer the subject fee to the shares subject's address.
Final Check:
require(success1 && success2, "Unable to send funds");
This final line checks whether both fee transfers were successful. If any of them fails, it raises an error.
🚨Remarkably for buyShares function from the check payment state:

require(msg.value >= price + protocolFee + subjectFee, "Insufficient payment");

If the buyer sends more Ether (msg.value)than the total cost of the transaction (i.e., price + protocolFee + subjectFee), the contract does not include a mechanism to refund the excess Ether back to the buyer.

The sellShares function:
```
contract FriendtechSharesV1 is Ownable {
    // (...SNIPPET...)
    
    function sellShares(address sharesSubject, uint256 amount) public payable {
        uint256 supply = sharesSupply[sharesSubject];
        require(supply > amount, "Cannot sell the last share");
        uint256 price = getPrice(supply - amount, amount);
        uint256 protocolFee = price * protocolFeePercent / 1 ether;
        uint256 subjectFee = price * subjectFeePercent / 1 ether;
        require(sharesBalance[sharesSubject][msg.sender] >= amount, "Insufficient shares");
        sharesBalance[sharesSubject][msg.sender] = sharesBalance[sharesSubject][msg.sender] - amount;
        sharesSupply[sharesSubject] = supply - amount;
        emit Trade(msg.sender, sharesSubject, false, amount, price, protocolFee, subjectFee, supply - amount);
        (bool success1, ) = msg.sender.call{value: price - protocolFee - subjectFee}("");
        (bool success2, ) = protocolFeeDestination.call{value: protocolFee}("");
        (bool success3, ) = sharesSubject.call{value: subjectFee}("");
        require(success1 && success2 && success3, "Unable to send funds");
    }
    
    // (...SNIPPET...)
}
```
The sellShares function enables users to sell their shares back to the contract. This function is essential for liquidity on the platform, allowing users to exit their share positions when desired. It involves several steps, including checking eligibility, calculating prices, handling fees, and updating balances.

Let’s break it down line by line:

Retrieve Supply:
uint256 supply = sharesSupply[sharesSubject];
This line retrieves the current supply of shares for the specified shares subject.
Check Validity:
require(supply > amount, “Cannot sell the last share”);
This line ensures that the user cannot sell more shares than what is available in the supply. If the supply is less than or equal to the amount, the transaction is halted with the specified error message.
Calculate Price:
uint256 price = getPrice(supply — amount, amount);
The price of the shares to be sold is calculated using the getPrice function. It considers the adjusted supply after selling the specified amount of shares.
Calculate Fees:
uint256 protocolFee = price * protocolFeePercent / 1 ether;
uint256 subjectFee = price * subjectFeePercent / 1 ether;
These lines calculate the protocol and subject fees based on the price of the shares to be sold.
Check Share Balance:
require(sharesBalance[sharesSubject][msg.sender] >= amount, “Insufficient shares”);
This line verifies that the user has a sufficient balance of shares to sell. If the user’s balance is less than the amount they intend to sell, the transaction will be reversed.
Update User Balance:
sharesBalance[sharesSubject][msg.sender] = sharesBalance[sharesSubject][msg.sender] — amount;
The function deducts the sold shares from the user’s balance.
Update Supply:
sharesSupply[sharesSubject] = supply — amount;
The total supply of shares for the shares subject is updated by reducing the sold amount.
Emit Trade Event:
emit Trade(msg.sender, sharesSubject, false, amount, price, protocolFee, subjectFee, supply — amount);
It emits an event with details of the trade, including the seller, shares subject, trade type (sell), the amount sold, price, fees, and supply.
Transfer Funds:
(bool success1, ) = msg.sender.call{value: price — protocolFee — subjectFee}(“”);
(bool success2, ) = protocolFeeDestination.call{value: protocolFee}(“”);
(bool success3, ) = sharesSubject.call{value: subjectFee}(“”);
These lines attempt to transfer the proceeds to the seller, protocol fees to the protocol fee destination, and subject fees to the subject’s address.
Final Check:
require(success1 && success2 && success3, “Unable to send funds”);
This line ensures that all fund transfers were successful. If any of them fail, the transaction is halted, preventing funds from being lost.

## Key enhancements in the Curves protocol include:

Token Export to ERC20: This pivotal feature allows users to transfer their tokens from the Curves protocol to the ERC20 format. Such interoperability significantly expands usability across various platforms. Within Curves, tokens lack decimal places, but when converted to ERC20, they adopt a standard 18-decimal format. Importantly, users can seamlessly reintegrate their ERC20 tokens into the Curves ecosystem, albeit only as whole, integer units.

Referral Fee Implementation: Curves empowers protocols built upon its framework by enabling them to earn a percentage of all user transaction fees. This incentive mechanism benefits both the base protocol and its derivative platforms.

Presale Feature: Learning from the pitfalls of friend.tech, particularly issues with frontrunners during token launches, Curves incorporates a presale phase. This allows creators to manage and stabilize their tokens prior to public trading, ensuring a more controlled and equitable distribution.

Token Holder Fee: To encourage long-term holding over short-term trading, Curves introduces a fee distribution model that rewards token holders. This fee is proportionally divided among all token holders, incentivizing sustained investment in the ecosystem.

![Imgur](https://i.imgur.com/Jw0ec4p.png)

## Codebase strenghs

- Codebase is well written and respects the solidity best practices.
- Codebase is fully decentralized, this is awesome as it erases any potential centralization risk.
- Comments and natspec were really helpfull, they were really detailled and self-explanatory.
- Redondants codes are refactored into modifiers.
- Documentation is well written and self-explanatory.
- Codebase was forked from an already strong codebase which makes it even harder to find any issue.

## Codebase weaknesses

- Codebase is not upgradable, this makes it harder to maintain in case of any security issue in the future

## Audits approach

- Day 1: read the docs and understand the protocol.
- Day 2: Delve deep into the codebase to get a general code architecture understanding.
- Day 3: Delve more deeper into friend.tech smarts contracts to understand more the code.
- Day 4: Compile the findings and write the report.






### Time spent:
22 hours
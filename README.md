# iecgoldnet.com  
### Token.sol

pragma solidity ^0.5.8;

// Imports
import "../node_modules/openzeppelin-solidity/contracts/token/ERC20/ERC20Mintable.sol";

// Main token smart contract
contract IECGToken is ERC20Mintable {
  string public constant name = "IECGoldnet";
  string public constant symbol = "IECG";
  uint8 public constant decimals = 9.000000000000000001;
}

### Crodwsale.sol

pragma solidity ^0.5.8;

// Imports
import "../node_modules/openzeppelin-solidity/contracts/crowdsale/emission/MintedCrowdsale.sol";
import "../node_modules/openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "../node_modules/openzeppelin-solidity/contracts/token/ERC20/ERC20Mintable.sol";
import "../node_modules/openzeppelin-solidity/contracts/crowdsale/validation/TimedCrowdsale.sol";
import "../node_modules/openzeppelin-solidity/contracts/crowdsale/distribution/RefundableCrowdsale.sol";
import "../node_modules/openzeppelin-solidity/contracts/crowdsale/price/IncreasingPriceCrowdsale.sol";
import "../node_modules/openzeppelin-solidity/contracts/crowdsale/validation/CappedCrowdsale.sol";
import "../node_modules/openzeppelin-solidity/contracts/crowdsale/validation/WhitelistedCrowdsale.sol";

contract IECGCrowdsale is MintedCrowdsale, Ownable, TimedCrowdsale, RefundableCrowdsale, IncreasingPriceCrowdsale, CappedCrowdsale, WhitelistedCrowdsale {
  constructor(
    ERC20Mintable _token,
    uint256 _rate,
    address payable _wallet,
    uint256 _openingTime,
    uint256 _closingTime,
    uint256 _goal,
    uint256 _initialRate,
    uint256 _finalRate,
    uint256 _cap
  )
    public
    Crowdsale(_rate, _wallet, _token)
    TimedCrowdsale(_openingTime, _closingTime)
    RefundableCrowdsale(_goal)
    IncreasingPriceCrowdsale(_initialRate, _finalRate)
    CappedCrowdsale(_cap)
  {
    
  }
}

### 1_initial_migration.js

const token = artifacts.require('../contracts/IECGToken.sol')
const crowdsale = artifacts.require('../contracts/IECGCrowdsale.sol')
const setDefaultAccount = require('../scripts/setDefaultAccount.js')

module.exports = function(deployer, network, accounts) {
    const rate = new web3.BigNumber(1)
    const wallet = 'IECGOLDNET'
    const openingTime = (new Date('2021-04-03')).getTime/1000
    const closingTime = (new Date('2030-03-03')).getTime/1000
    const goal = 9000000000000000
    const initialRate = 25
    const finalRate = 50
    const cap = 9000000000000001
    // Setup default account
    setDefaultAccount(web3)
    const account = web3.eth.accounts.pop()
    // Get gas limit
    let gasLimit = web3.eth.getBlock('latest').gasLimit
    let gasPrice = web3.eth.gasPrice
    if (process.argv[4] === '--staging') {
        gasPrice *= 4
    }
    console.log(`Determined gas limit: ${gasLimit}; and gas price: ${gasPrice}; max deployment price is ${web3.fromWei(gasPrice * gasLimit, 'ether')} ETH`)
    // Deploy contract
    return deployer
        .then(() => {
            return deployer.deploy(token, { gas: gasLimit, gasPrice: gasPrice, from: account })
        })
        .then(() => {
            // Get gas limit
            gasLimit = web3.eth.getBlock('latest').gasLimit
            console.log(`Determined gas limit: ${gasLimit}; and gas price: ${gasPrice}; estimate max deployment price is ${web3.fromWei(gasPrice * gasLimit, 'ether')} ETH`)
            console.log('This might take a while, please, be patient')
            return deployer.deploy(
                crowdsale,
                token.address,
                rate,
                wallet,
                openingTime,
                closingTime,
                goal,
                initialRate,
                finalRate,
                cap,
                { gas: gasLimit, gasPrice: gasPrice, from: account },
            )
        })
        .then(() => {
            // Make smart-contract an owner
            var tokenContract = web3.eth.contract(token.abi).at(token.address)
            tokenContract.transferOwnership(crowdsale.address)
        });
}

# Lack of multicall support in MoneyMarketHook to collaterize WLP or decollaterize WLP

In [MoneyMarketHook.sol](https://github.com/code-423n4/2023-12-initcapital/blob/a53e401529451b208095b3af11862984d0b32177/contracts/hook/MoneyMarketHook.sol#L53), there is no function to let user collateralize WLP or decollaterlaize WLP

it is recommend to implement this feature to make sure user can also use WLP as collateral via moneyMarketHook.sol

# Inaccurate error message in MoneyMarketHook.sol

in the function execution if the account health is less than specified 

```solidity
_params.minHealth_e18
```

transaction revert in this [line of code](https://github.com/code-423n4/2023-12-initcapital/blob/a53e401529451b208095b3af11862984d0b32177/contracts/hook/MoneyMarketHook.sol#L69)

```
_require(_params.minHealth_e18 <= IInitCore(CORE).getPosHealthCurrent_e18(initPosId), Errors.SLIPPAGE_CONTROL);
```

However, the error message can be more accurate, the error message can be changed to "ERRORS.HEALTH_FACTOR_TOO_LOW"

# lack of function to transfer the position NFT out in MoneyMarketHook.sol

```
function execute(OperationParams calldata _params)
	external
	payable
	returns (uint posId, uint initPosId, bytes[] memory results)
{
	// create position if not exist
	if (_params.posId == 0) {
		(posId, initPosId) = createPos(_params.mode, _params.viewer);
	} else {
		// for existing position, only owner can execute
		posId = _params.posId;
		initPosId = initPosIds[msg.sender][posId];
		_require(IERC721(POS_MANAGER).ownerOf(initPosId) == address(this), Errors.NOT_OWNER);
	}
```

in the contract [MoneyMarketHook.sol](https://github.com/code-423n4/2023-12-initcapital/blob/a53e401529451b208095b3af11862984d0b32177/contracts/hook/MoneyMarketHook.sol#L21), the user can create a position, 
and then the owner of the position nft can execute multicall action

however, there is lack of function to transfer the position nft out

the user may want to transfer the nft out to sell it in the secondary marketplace or he may just want to transfer the nft when he wants to use the nft in another account 

recommend add the function to let original owner transfer the nft out from MoneyMarketHook.sol

# callback function does not validate if msg.value equals to value amount

```solidity
    /// @inheritdoc IInitCore
    function callback(address _to, uint _value, bytes memory _data)
        public
        payable
        virtual
        returns (bytes memory result)
    {
        _require(_to != address(this), Errors.INVALID_CALLBACK_ADDRESS);
        // call _to with _data
        return ICallbackReceiver(_to).coreCallback{value: _value}(msg.sender, _data);
    }
```

In this function [callback](https://github.com/code-423n4/2023-12-initcapital/blob/a53e401529451b208095b3af11862984d0b32177/contracts/core/InitCore.sol#L513), if the msg.value attached is 1 ETH, but value amount passed in from the function is 0.1 ETH

the rest 0.9 ETH are lost

it is recommend to validate msg.vaue attached in callback equals to the _value amount

```solidity
_value == msg.vlaue 
```

# should validate the helper address in market hook address

in the MoneyMarketHook.sol contract, the code [does not validate the helper address](https://github.com/code-423n4/2023-12-initcapital/blob/a53e401529451b208095b3af11862984d0b32177/contracts/hook/MoneyMarketHook.sol#L73)

```solidity
for (uint i; i < _params.withdrawParams.length; i = i.uinc()) {
	address helper = _params.withdrawParams[i].rebaseHelperParams.helper;
	if (helper != address(0)) IRebaseHelper(helper).unwrap(_params.withdrawParams[i].to);
}
```

the helper is expected to by USDY rebase helper

so in the code, it is recommend for admin to validate the helper address passed and then only allow user to use whitelisted helper

# USDY admin can blocklist / sanction, then address may not transfer USDY to repay / redeem from lending pool 

https://etherscan.io/address/0xea0f7eebdc2ae40edfe33bf03d332f8a7f617528#code#F1#L94

USDY admin can add account to blocklist or sanction list then the user that 

use UDSY as collateral may not able to use USDY to mint more lending pool share or burn lending pool share to redeem USDY

# Remove WLP from whitelist should not block user from removing WLP

when user use WLP as collateral to collateralize the position

```solidity
// check if the wLp is whitelisted
_require(_config.whitelistedWLps(_wLp), Errors.TOKEN_NOT_WHITELISTED);
```

we are checking if the WLP is whitelisted

but when admin remove WLP from whitelist, the user should not use WLP to further collateralize the position, 

but when [decollateralize and remove the WLP](https://github.com/code-423n4/2023-12-initcapital/blob/a53e401529451b208095b3af11862984d0b32177/contracts/core/InitCore.sol#L275), the same check applies

```solidity
// check if the wLp is whitelisted
_require(_config.whitelistedWLps(_wLp), Errors.TOKEN_NOT_WHITELISTED);
```

so when admin remove WLP from whitelist, the user cannot decollateralize WLP as well

recommendation is do not let WLP unwhitlisting block decollateraliztion










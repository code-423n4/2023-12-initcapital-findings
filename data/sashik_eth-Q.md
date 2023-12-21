## L1 - Minting of new pool shares could be DOSed

The `InitCore#mintTo` function checks that total assets are less than the pool's supply cap and reverts in other case. This check could be used in the sandwich attack to prevent users from minting new pool shares:
```solidity
File: InitCore.sol
100:     function mintTo(address _pool, address _to) public virtual nonReentrant returns (uint shares) {
...
107:         _require(ILendingPool(_pool).totalAssets() <= poolConfig.supplyCap, Errors.SUPPLY_CAP_REACHED); 
```

## L2 - Borrowing could be DOSed

Similar to the previous issue borrowing function could be DOS due to check in the `updateModeDebtShares` function:
```solidity
File: RiskManager.sol
70:     function updateModeDebtShares(uint16 _mode, address _pool, int _deltaShares) external onlyCore {
...
75:             _require(currentDebt <= debtCeilingInfo.ceilAmt, Errors.DEBT_CEILING_EXCEEDED);
``` 

## L3 - Repay function emit the wrong amount of shares

At line 538 amount of shares that would be prepared is chosen between the specified amount and actual position shares. However, event at line 550 always emits a specified amount:
```solidity
File: InitCore.sol
530:     function _repay(IConfig _config, uint16 _mode, uint _posId, address _pool, uint _shares)
531:         internal
532:         returns (address tokenToRepay, uint amt)
533:     {
...
538:         uint sharesToRepay = _shares < positionDebtShares ? _shares : positionDebtShares;
...
550:         emit Repay(_pool, _posId, msg.sender, _shares, amt); 
```

## L4 - Mantle network does not support 'push0'

The protocol desired chain is a Mantle network, and contracts pragma allows to use of solidity versions `^0.8.19`. At the same time due to the Mantle docs, it's not support versions 0.8.20 and older:

https://docs.mantle.xyz/network/for-devs/solidity-support

## L5 - Previous treasury could miss part of the fee

The `setTreasury` function lacks the `accrue` modifier. This could lead to the situation when the governor change treasuty address and fees that should be accrued to the current treasury address would be minted to the new one after the next `accrueInterest` call.

```solidity
File: LendingPool.sol
245:     function setTreasury(address _treasury) external onlyGovernor {
246:         treasury = _treasury;
247:         emit SetTreasury(_treasury);
248:     }
```

## L6 - Wrong error code

`liquidate` function would revert with error code `TOKEN_NOT_WHITELISTED` while it should revert `INVALID_MODE` in case if specified pool is not allowed as a collateral for the specified mode:

```solidity
File: InitCore.sol
282:     function liquidate(uint _posId, address _poolToRepay, uint _repayShares, address _poolOut, uint _minShares)
283:         public
284:         virtual
285:         nonReentrant
286:         returns (uint shares)
287:     {
...
290:         _require(vars.config.isAllowedForCollateral(vars.mode, _poolOut), Errors.TOKEN_NOT_WHITELISTED); // config and mode are already stored
291: 
```

## L7 - Protocol would not work with tokens that not realise `decimals` function

Due to ERC20 standard `decimals` function is optional, this could cause problems for protocol that expect tokens to return correct value for `decimals` call:

```solidity
File: LendingPool.sol
93:     // functions
94:     /// @inheritdoc ERC20Upgradeable
95:     function decimals() public view override returns (uint8) {
96:         return IERC20Metadata(underlyingToken).decimals(); 
97:     }

File: Api3OracleReader.sol
74:     function getPrice_e36(address _token) external view returns (uint price_e36) {
...
80:         // get price and token's decimals
81:         uint decimals = uint(IERC20Metadata(_token).decimals()); 
```

## N1 - Unused imported libraries 

```solidity
File: LiqIncentiveCalculator.sol
6: import {LogExpMath} from '../common/library/LogExpMath.sol'; 

File: Config.sol
4: import '../common/library/InitErrors.sol';

File: InitCore.sol
5: import '../common/library/InitErrors.sol';

File: InitCore.sol
11: import {ModeConfig, PoolConfig, TokenFactors, ModeStatus, IConfig} from '../interfaces/core/IConfig.sol';

File: InitCore.sol
25: import {IERC721} from '@openzeppelin-contracts/token/ERC721/IERC721.sol';

File: PosManager.sol
4: import '../common/library/InitErrors.sol';

File: BaseRebaseHelper.sol
4: import '../../common/library/InitErrors.sol';

File: MoneyMarketHook.sol
4: import '../common/library/InitErrors.sol';

File: LendingPool.sol
4: import '../common/library/InitErrors.sol';

File: Api3OracleReader.sol
8: import '../common/library/InitErrors.sol';

File: InitOracle.sol
4: import '../common/library/InitErrors.sol';

File: RiskManager.sol
6: import '../common/library/InitErrors.sol';
```

## N2 - Missing NatSpec for _maxCollCount parameter
```solidity
File: PosManager.sol
63:     // initializer
64:     /// @dev initialize the contract, set the ERC721's name and symbol, and set the init core address
65:     /// @param _name ERC721's name
66:     /// @param _symbol ERC721's symbol
67:     /// @param _core core address
68:     function initialize(string calldata _name, string calldata _symbol, address _core, uint8 _maxCollCount) 
```

## N3 - syncCash modifiers order inconsistency 

The `syncCash` function has a different order of modifiers compared to other functions in the contract:
```solidity
File: LendingPool.sol
136:     function repay(uint _shares) external onlyCore accrue returns (uint amt) {
...
148:     function syncCash() external accrue onlyCore returns (uint newCash) { 
```


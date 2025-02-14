The LP Token and Liquidity Pool for two-coin cryptoswap pools are separated.

The LP Token is generated using the `token_implementation` contract within the [Factory](https://etherscan.io/address/0xf18056bbd320e96a48e3fbf8bc061322531aac99) when the `deploy_pool()` function is called. Subsequently, the token is set up by calling the `initialize()` function, which sets its name, symbol, and associated liquidity pool.

```shell
>>> Factory.token_implementation()
'0xc08550A4cc5333f40e593eCc4C4724808085D304'
```


??? quote "deploy_pool()"

    ```python hl_lines="2 56 59"
    @external
    def deploy_pool(
        _name: String[32],
        _symbol: String[10],
        _coins: address[2],
        A: uint256,
        gamma: uint256,
        mid_fee: uint256,
        out_fee: uint256,
        allowed_extra_profit: uint256,
        fee_gamma: uint256,
        adjustment_step: uint256,
        admin_fee: uint256,
        ma_half_time: uint256,
        initial_price: uint256
    ) -> address:
        """
        @notice Deploy a new pool
        @param _name Name of the new plain pool
        @param _symbol Symbol for the new plain pool - will be concatenated with factory symbol
        Other parameters need some description
        @return Address of the deployed pool
        """
        # Validate parameters
        assert A > MIN_A-1
        assert A < MAX_A+1
        assert gamma > MIN_GAMMA-1
        assert gamma < MAX_GAMMA+1
        assert mid_fee > MIN_FEE-1
        assert mid_fee < MAX_FEE-1
        assert out_fee >= mid_fee
        assert out_fee < MAX_FEE-1
        assert admin_fee < 10**18+1
        assert allowed_extra_profit < 10**16+1
        assert fee_gamma < 10**18+1
        assert fee_gamma > 0
        assert adjustment_step < 10**18+1
        assert adjustment_step > 0
        assert ma_half_time < 7 * 86400
        assert ma_half_time > 0
        assert initial_price > 10**6
        assert initial_price < 10**30
        assert _coins[0] != _coins[1], "Duplicate coins"

        decimals: uint256[2] = empty(uint256[2])
        for i in range(2):
            d: uint256 = ERC20(_coins[i]).decimals()
            assert d < 19, "Max 18 decimals for coins"
            decimals[i] = d
        precisions: uint256 = (18 - decimals[0]) + shift(18 - decimals[1], 8)


        name: String[64] = concat("Curve.fi Factory Crypto Pool: ", _name)
        symbol: String[32] = concat(_symbol, "-f")

        token: address = create_forwarder_to(self.token_implementation)
        pool: address = create_forwarder_to(self.pool_implementation)

        Token(token).initialize(name, symbol, pool)
        CryptoPool(pool).initialize(
            A, gamma, mid_fee, out_fee, allowed_extra_profit, fee_gamma,
            adjustment_step, admin_fee, ma_half_time, initial_price,
            token, _coins, precisions)

        length: uint256 = self.pool_count
        self.pool_list[length] = pool
        self.pool_count = length + 1
        self.pool_data[pool].token = token
        self.pool_data[pool].decimals = shift(decimals[0], 8) + decimals[1]
        self.pool_data[pool].coins = _coins

        key: uint256 = bitwise_xor(convert(_coins[0], uint256), convert(_coins[1], uint256))
        length = self.market_counts[key]
        self.markets[key][length] = pool
        self.market_counts[key] = length + 1

        log CryptoPoolDeployed(
            token, _coins,
            A, gamma, mid_fee, out_fee, allowed_extra_profit, fee_gamma,
            adjustment_step, admin_fee, ma_half_time, initial_price,
            msg.sender)
        return pool
    ```


## **LP Token Info Methods**

### `name`
!!! description "`LPToken.name() -> String[64]: view`"

    Getter for the name of the LP token.

    Returns: name (`String[64]`).

    ??? quote "Source code"

        ```python hl_lines="1 4 7"
        name: public(String[64])

        @external
        def initialize(_name: String[64], _symbol: String[32], _pool: address):
            assert self.minter == ZERO_ADDRESS  # dev: check that we call it from factory

            self.name = _name
            self.symbol = _symbol
            self.minter = _pool

            self.DOMAIN_SEPARATOR = keccak256(
                _abi_encode(EIP712_TYPEHASH, keccak256(_name), keccak256(VERSION), chain.id, self)
            )

            # fire a transfer event so block explorers identify the contract as an ERC20
            log Transfer(ZERO_ADDRESS, msg.sender, 0)
        ```

    === "Example"

        ```shell
        >>> LPToken.name()
        'Curve.fi Factory Crypto Pool: LDO/ETH'
        ```


### `symbol`
!!! description "`LPToken.symbol() -> String[32]: view`"

    Getter for the symbol of the LP token.

    Returns: symbol (`String[32]`).

    ??? quote "Source code"

        ```python hl_lines="1 4 8"
        symbol: public(String[32])

        @external
        def initialize(_name: String[64], _symbol: String[32], _pool: address):
            assert self.minter == ZERO_ADDRESS  # dev: check that we call it from factory

            self.name = _name
            self.symbol = _symbol
            self.minter = _pool

            self.DOMAIN_SEPARATOR = keccak256(
                _abi_encode(EIP712_TYPEHASH, keccak256(_name), keccak256(VERSION), chain.id, self)
            )

            # fire a transfer event so block explorers identify the contract as an ERC20
            log Transfer(ZERO_ADDRESS, msg.sender, 0)
        ```

    === "Example"

        ```shell
        >>> LPToken.symbol()
        'LDOETH-f'
        ```


### `decimals`
!!! description "`LPToken.decimals() -> uint8`"

    Getter for the decimals of the LP token.

    Returns: decimals (`uint8`).

    ??? quote "Source code"

        ```python hl_lines="3"
        @view
        @external
        def decimals() -> uint8:
            """
            @notice Get the number of decimals for this token
            @dev Implemented as a view method to reduce gas costs
            @return uint8 decimal places
            """
            return 18
        ```

    === "Example"

        ```shell
        >>> LPToken.decimals()
        18
        ```


### `version`
!!! description "`LPToken.version() -> String[8]:`"

    Getter for the version of the LP token.

    Returns: version (`String[8]`).

    ??? quote "Source code"

        ```python hl_lines="1 5 9"
        VERSION: constant(String[8]) = "v5.0.0"

        @view
        @external
        def version() -> String[8]:
            """
            @notice Get the version of this token contract
            """
            return VERSION
        ```

    === "Example"

        ```shell
        >>> LPToken.version()
        'v5.0.0'
        ```


### `balanceOf`
!!! description "`LPToken.balanceOf(arg0: address) -> uint256: view`"

    Getter for the LP token balance of an address.

    Returns: token balance (`uint256`).

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `arg0` |  `address` | Address to get the balance for |

    ??? quote "Source code"

        ```python hl_lines="1"
        balanceOf: public(HashMap[address, uint256])
        ```

    === "Example"

        ```shell
        >>> LPToken.balanceOf("0xe5d5Aa1Bbe72F68dF42432813485cA1Fc998DE32")
        74284034901658384235023
        ```


### `totalSupply`
!!! description "`LPToken.totalSupply() -> uint256: view`"

    Getter for the total supply of the LP token.

    Returns: total supply (`uint256`).

    ??? quote "Source code"

        ```python hl_lines="1"
        totalSupply: public(uint256)
        ```

    === "Example"

        ```shell
        >>> LPToken.totalSupply()
        74357443715423884544842
        ```


### `minter`
!!! description "`LPToken.totalSupply() -> uint256: view`"

    Getter for the minter contract of the LP token. Minter contract address is the liquidity pool itself.

    Returns: minter (`address`).

    ??? quote "Source code"

        ```python hl_lines="1 4 5 8 9"
        minter: public(address)

        @external
        def __init__():
            self.minter = 0x0000000000000000000000000000000000000001

        @external
        def initialize(_name: String[64], _symbol: String[32], _pool: address):
            assert self.minter == ZERO_ADDRESS  # dev: check that we call it from factory

            self.name = _name
            self.symbol = _symbol
            self.minter = _pool

            self.DOMAIN_SEPARATOR = keccak256(
                _abi_encode(EIP712_TYPEHASH, keccak256(_name), keccak256(VERSION), chain.id, self)
            )

            # fire a transfer event so block explorers identify the contract as an ERC20
            log Transfer(ZERO_ADDRESS, msg.sender, 0)
        ```

    === "Example"

        ```shell
        >>> LPToken.minter()
        '0x9409280DC1e6D33AB7A8C6EC03e5763FB61772B5'
        ```



## **Allowance and Transfer Methods**
### `transfer`
!!! description "`LPToken.transfer(_to: address, _value: uint256) -> bool`"

    Function to transfer `_value` token from `msg.sender` to `_to`.

    Returns: True (`bool`).

    Emits: `Transfer`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_to` |  `address` | Address to transfer to |
    | `_value` |  `uint256` | Amount to transfer |

    ??? quote "Source code"

        ```python hl_lines="1 7 18 19"
        event Transfer:
            _from: indexed(address)
            _to: indexed(address)
            _value: uint256

        @external
        def transfer(_to: address, _value: uint256) -> bool:
            """
            @dev Transfer token for a specified address
            @param _to The address to transfer to.
            @param _value The amount to be transferred.
            """
            # NOTE: vyper does not allow underflows
            #       so the following subtraction would revert on insufficient balance
            self.balanceOf[msg.sender] -= _value
            self.balanceOf[_to] += _value

            log Transfer(msg.sender, _to, _value)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.transfer("0xbabe61887f1de2713c6f97e567623453d3C79f67", 100)
        True
        ```


### `transferFrom`
!!! description "`LPToken.transfer(_to: address, _value: uin256) -> bool:`"

    Function to transfer `_value` token from `msg.sender` to `_to`. Needs [`allowance`](#allowance) to successfully transfer on behalf of someone else.

    Returns: True or False (`bool`).

    Emits: `Transfer`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_to` |  `address` | Address to transfer to |
    | `_value` |  `uint256` | Amount to transfer |

    ??? quote "Source code"

        ```python hl_lines="1 7 21 22"
        event Transfer:
            _from: indexed(address)
            _to: indexed(address)
            _value: uint256

        @external
        def transferFrom(_from: address, _to: address, _value: uint256) -> bool:
            """
            @dev Transfer tokens from one address to another.
            @param _from address The address which you want to send tokens from
            @param _to address The address which you want to transfer to
            @param _value uint256 the amount of tokens to be transferred
            """
            self.balanceOf[_from] -= _value
            self.balanceOf[_to] += _value

            _allowance: uint256 = self.allowance[_from][msg.sender]
            if _allowance != MAX_UINT256:
                self.allowance[_from][msg.sender] = _allowance - _value

            log Transfer(_from, _to, _value)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.transferFrom("0xbabe61887f1de2713c6f97e567623453d3C79f67", 100)
        True
        ```


### `approve`
!!! description "`LPToken.approve(_spender: address, _value: uint256) -> bool:`"

    Function to approve `_spender` to transfer `_value` on behalf of msg.sender.

    Returns: True (`bool`).

    Emits: `Approval`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_spender` |  `address` | Address approved to spend funds |
    | `_value` |  `uint256` | Amount of tokens allowed to spend |

    ??? quote "Source code"

        ```python hl_lines="1 7 22 23"
        event Approval:
            _owner: indexed(address)
            _spender: indexed(address)
            _value: uint256

        @external
        def approve(_spender: address, _value: uint256) -> bool:
            """
            @notice Approve the passed address to transfer the specified amount of
                    tokens on behalf of msg.sender
            @dev Beware that changing an allowance via this method brings the risk
                that someone may use both the old and new allowance by unfortunate
                transaction ordering. This may be mitigated with the use of
                {increaseAllowance} and {decreaseAllowance}.
                https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
            @param _spender The address which will transfer the funds
            @param _value The amount of tokens that may be transferred
            @return bool success
            """
            self.allowance[msg.sender][_spender] = _value

            log Approval(msg.sender, _spender, _value)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.approve("todo")
        "todo"
        ```


### `permit`
!!! description "`LPToken.permit(_owner: address, _spender: address, _value: uint256, _deadline: uint256, _v: uint8, _r: bytes32, _s: bytes32) -> bool::`"

    Function to approve the spender by the owner's signature to expend the owner's tokens.

    Returns: True (`bool`).

    Emits: `Approval`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_to` |  `address` | Address to transfer to |
    | `_value` |  `uint256` | Amount to transfer |

    ??? quote "Source code"

        ```python hl_lines="1 7 "
        event Approval:
            _owner: indexed(address)
            _spender: indexed(address)
            _value: uint256

        @external
        def permit(
            _owner: address,
            _spender: address,
            _value: uint256,
            _deadline: uint256,
            _v: uint8,
            _r: bytes32,
            _s: bytes32
        ) -> bool:
            """
            @notice Approves spender by owner's signature to expend owner's tokens.
                See https://eips.ethereum.org/EIPS/eip-2612.
            @dev Inspired by https://github.com/yearn/yearn-vaults/blob/main/contracts/Vault.vy#L753-L793
            @dev Supports smart contract wallets which implement ERC1271
                https://eips.ethereum.org/EIPS/eip-1271
            @param _owner The address which is a source of funds and has signed the Permit.
            @param _spender The address which is allowed to spend the funds.
            @param _value The amount of tokens to be spent.
            @param _deadline The timestamp after which the Permit is no longer valid.
            @param _v The bytes[64] of the valid secp256k1 signature of permit by owner
            @param _r The bytes[0:32] of the valid secp256k1 signature of permit by owner
            @param _s The bytes[32:64] of the valid secp256k1 signature of permit by owner
            @return True, if transaction completes successfully
            """
            assert _owner != ZERO_ADDRESS
            assert block.timestamp <= _deadline

            nonce: uint256 = self.nonces[_owner]
            digest: bytes32 = keccak256(
                concat(
                    b"\x19\x01",
                    self.DOMAIN_SEPARATOR,
                    keccak256(_abi_encode(PERMIT_TYPEHASH, _owner, _spender, _value, nonce, _deadline))
                )
            )

            if _owner.is_contract:
                sig: Bytes[65] = concat(_abi_encode(_r, _s), slice(convert(_v, bytes32), 31, 1))
                # reentrancy not a concern since this is a staticcall
                assert ERC1271(_owner).isValidSignature(digest, sig) == ERC1271_MAGIC_VAL
            else:
                assert ecrecover(digest, convert(_v, uint256), convert(_r, uint256), convert(_s, uint256)) == _owner

            self.allowance[_owner][_spender] = _value
            self.nonces[_owner] = nonce + 1

            log Approval(_owner, _spender, _value)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.permit()
        "todo"
        ```


### `allowance`
!!! description "`LPToken.allowance(arg0: address, arg1: address) -> uint256: view`"

    Getter method to check the allowance of `arg0` for funds of `arg1`.

    Returns: allowed amount (`uint256`).

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `arg0` |  `address` | Address of the spender |
    | `arg0` |  `address` | Address to the token owner |

    ??? quote "Source code"

        ```python hl_lines="1"
        allowance: public(HashMap[address, HashMap[address, uint256]])
        ```

    === "Example"

        ```shell
        >>> LPToken.allowance("todo")
        "todo"
        ```


### `increaseAllowance`
!!! description "`LPToken.increaseAllowance(_spender: address, _added_value: uint256) -> bool:`"

    Function to increase the allowance granted to `_spender`.

    Returns: True or False (`bool`).

    Emits: `Approval`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_spender` |  `address` | Address to increase the allowance of  |
    | `_added_value` |  `uint256` | Amount ot increase the allowance by |

    ??? quote "Source code"

        ```python hl_lines="1 6 9 21 22"
        event Approval:
            _owner: indexed(address)
            _spender: indexed(address)
            _value: uint256

        allowance: public(HashMap[address, HashMap[address, uint256]])

        @external
        def increaseAllowance(_spender: address, _added_value: uint256) -> bool:
            """
            @notice Increase the allowance granted to `_spender` by the caller
            @dev This is alternative to {approve} that can be used as a mitigation for
                the potential race condition
            @param _spender The address which will transfer the funds
            @param _added_value The amount of to increase the allowance
            @return bool success
            """
            allowance: uint256 = self.allowance[msg.sender][_spender] + _added_value
            self.allowance[msg.sender][_spender] = allowance

            log Approval(msg.sender, _spender, allowance)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.increaseAllowance("todo")
        "todo"
        ```


### `decreaseAllowance`
!!! description "`LPToken.decreaseAllowance(_spender: address, _subtracted_value: uint256) -> bool:`"

    Function to decrease the allowance granted to `_spender`.

    Returns: True or False (`bool`).

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_spender` |  `address` | Address to decrease the allowance of  |
    | `_subtracted_value` |  `uint256` | Amount ot decrease the allowance by |

    ??? quote "Source code"

        ```python hl_lines="1 6 9 21 22"
        event Approval:
            _owner: indexed(address)
            _spender: indexed(address)
            _value: uint256

        allowance: public(HashMap[address, HashMap[address, uint256]])

        @external
        def decreaseAllowance(_spender: address, _subtracted_value: uint256) -> bool:
            """
            @notice Decrease the allowance granted to `_spender` by the caller
            @dev This is alternative to {approve} that can be used as a mitigation for
                the potential race condition
            @param _spender The address which will transfer the funds
            @param _subtracted_value The amount of to decrease the allowance
            @return bool success
            """
            allowance: uint256 = self.allowance[msg.sender][_spender] - _subtracted_value
            self.allowance[msg.sender][_spender] = allowance

            log Approval(msg.sender, _spender, allowance)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.allowance("todo")
        "todo"
        ```


## **Minting and Burning**

LP Tokens are minted when users deposit funds into the liquidity pool. Upon calling the `add_liquidity` function on the pool, it triggers the `mint` function of the LP Token to mint the corresponding tokens.
When liquidity is withdrawn using `remove_liquidity`, the LP tokens are burned through the `burnFrom` method.

The logic for both minting and burning the tokens resides in the pool contract.


### `mint`
!!! description "`LPToken.mint(_to: address, _value: uint256) -> bool:`"

    !!!guard "Guarded Method"
        This function is only callable by the `minter` of the contract, which is the liquidity pool.

    Function to mint `_value` LP Tokens and transfer them to `_to`.

    Returns: True (`bool`).

    Emits: `Transfer`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_spender` |  `address` | Address to decrease the allowance of  |
    | `_subtracted_value` |  `uint256` | Amount ot decrease the allowance by |

    ??? quote "Source code"

        ```python hl_lines="1 7 20 21"
        event Transfer:
            _from: indexed(address)
            _to: indexed(address)
            _value: uint256

        @external
        def mint(_to: address, _value: uint256) -> bool:
            """
            @dev Mint an amount of the token and assigns it to an account.
                This encapsulates the modification of balances such that the
                proper events are emitted.
            @param _to The account that will receive the created tokens.
            @param _value The amount that will be created.
            """
            assert msg.sender == self.minter

            self.totalSupply += _value
            self.balanceOf[_to] += _value

            log Transfer(ZERO_ADDRESS, _to, _value)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.mint("todo")
        "todo"
        ```


### `burnFrom`
!!! description "`LPToken.burnFrom(_to: address, _value: uint256) -> bool:`"

    Function to burn `_value` LP Tokens from `_to` and transfer them to `ZERO_ADDRESS`.

    Returns: True (`bool`).

    Emits: `Transfer`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_spender` |  `address` | Address to decrease the allowance of  |
    | `_subtracted_value` |  `uint256` | Amount ot decrease the allowance by |

    ??? quote "Source code"

        ```python hl_lines="1 7 18 19"
        event Transfer:
            _from: indexed(address)
            _to: indexed(address)
            _value: uint256

        @external
        def burnFrom(_to: address, _value: uint256) -> bool:
            """
            @dev Burn an amount of the token from a given account.
            @param _to The account whose tokens will be burned.
            @param _value The amount that will be burned.
            """
            assert msg.sender == self.minter

            self.totalSupply -= _value
            self.balanceOf[_to] -= _value

            log Transfer(_to, ZERO_ADDRESS, _value)
            return True
        ```

    === "Example"

        ```shell
        >>> LPToken.burnFrom("todo")
        "todo"
        ```



## **Initialize Method**

The `initialize` function is used to initialize the pool when it's created through the factory.

### `initialize`
!!! description "`LPToken.initialize(_name: String[64], _symbol: String[32], _pool: address):`"

    Function to initialize the LP Token and setting name (`_name`), symbol (`_symbol`) and the corresponding liquidity pool (`_pool`).  
    This function triggers a transfer event, enabling block explorers to recognize the contract as an ERC20.

    Emits: `Transfer`

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_spender` |  `address` | Address to decrease the allowance of  |
    | `_subtracted_value` |  `uint256` | Amount ot decrease the allowance by |

    ??? quote "Source code"

        ```python hl_lines="1 7 19"
        event Transfer:
            _from: indexed(address)
            _to: indexed(address)
            _value: uint256

        @external
        def initialize(_name: String[64], _symbol: String[32], _pool: address):
            assert self.minter == ZERO_ADDRESS  # dev: check that we call it from factory

            self.name = _name
            self.symbol = _symbol
            self.minter = _pool

            self.DOMAIN_SEPARATOR = keccak256(
                _abi_encode(EIP712_TYPEHASH, keccak256(_name), keccak256(VERSION), chain.id, self)
            )

            # fire a transfer event so block explorers identify the contract as an ERC20
            log Transfer(ZERO_ADDRESS, msg.sender, 0)
        ```

    === "Example"

        ```shell
        >>> LPToken.initialize("todo")
        "todo"
        ```
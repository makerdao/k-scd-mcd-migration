DSToken
```k
syntax Int ::= "#DSToken.supply" [function]
rule #DSToken.supply => 0

syntax Int ::= "#DSToken.balances" "[" Int "]" [function]
rule #DSToken.balances[A] => #hashedLocation("Solidity", 1, A)

syntax Int ::= "#DSToken.allowance" "[" Int "][" Int "]" [function]
rule #DSToken.allowance[A][B] => #hashedLocation("Solidity", 2, A B)

syntax Int ::= "#DSToken.authority" [function]
rule #DSToken.authority => 3

syntax Int ::= "#DSToken.owner_stopped" [function]
rule #DSToken.owner_stopped => 4

syntax Int ::= "#DSToken.symbol" [function]
rule #DSToken.symbol => 5

syntax Int ::= "#DSToken.decimals" [function]
rule #DSToken.decimals => 6
```

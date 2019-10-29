Vat
```k
// act: public
syntax Int ::= "#Vat.wards" "[" Int "]" [function]
// -----------------------------------------------
// doc: whether `$0` is an owner of `Vat`
// act: address `$0` is `. == 1 ? authorised : unauthorised`
rule #Vat.wards[A] => #hashedLocation("Solidity", 0, A)

syntax Int ::= "#Vat.can" "[" Int "][" Int "]" [function]
// -----------------------------------------------
// doc: whether `$1` can spend the resources of `$0`
// act: address `$0` has authorized `$1`
rule #Vat.can[A][B] => #hashedLocation("Solidity", 1, A B)

syntax Int ::= "#Vat.ilks" "[" Int "].Art" [function]
// ----------------------------------------------------
// doc: total debt units issued from `$0`
// act: `$0` has debt issuance `.`
rule #Vat.ilks[Ilk].Art => #hashedLocation("Solidity", 2, Ilk) +Int 0

syntax Int ::= "#Vat.ilks" "[" Int "].rate" [function]
// ----------------------------------------------------
// doc: debt unit rate of `$0`
// act: `$0` has debt unit rate `.`
rule #Vat.ilks[Ilk].rate => #hashedLocation("Solidity", 2, Ilk) +Int 1

syntax Int ::= "#Vat.ilks" "[" Int "].spot" [function]
// -----------------------------------------------
// doc: price with safety margin for `$0`
// act: `$0` has safety margin `.`
rule #Vat.ilks[Ilk].spot => #hashedLocation("Solidity", 2, Ilk) +Int 2

syntax Int ::= "#Vat.ilks" "[" Int "].line" [function]
// -----------------------------------------------
// doc: debt ceiling for `$0`
// act: `$0` has debt ceiling `.`
rule #Vat.ilks[Ilk].line => #hashedLocation("Solidity", 2, Ilk) +Int 3

syntax Int ::= "#Vat.ilks" "[" Int "].dust" [function]
// -----------------------------------------------
// doc: urn debt floor for `$0`
// act: `$0` has debt floor `.`
rule #Vat.ilks[Ilk].dust => #hashedLocation("Solidity", 2, Ilk) +Int 4

syntax Int ::= "#Vat.urns" "[" Int "][" Int "].ink" [function]
// ----------------------------------------------------------
// doc: locked collateral units in `$0` assigned to `$1`
// act: agent `$1` has `.` collateral units in `$0`
rule #Vat.urns[Ilk][Usr].ink => #hashedLocation("Solidity", 3, Ilk Usr)

syntax Int ::= "#Vat.urns" "[" Int "][" Int "].art" [function]
// ----------------------------------------------------------
// doc: debt units in `$0` assigned to `$1`
// act: agent `$1` has `.` debt units in `$0`
rule #Vat.urns[Ilk][Usr].art => #hashedLocation("Solidity", 3, Ilk Usr) +Int 1

syntax Int ::= "#Vat.gem" "[" Int "][" Int "]" [function]
// ---------------------------------------------
// doc: unlocked collateral in `$0` assigned to `$1`
// act: agent `$1` has `.` unlocked collateral in `$0`
rule #Vat.gem[Ilk][Usr] => #hashedLocation("Solidity", 4, Ilk Usr)

syntax Int ::= "#Vat.dai" "[" Int "]" [function]
// ---------------------------------------------
// doc: dai assigned to `$0`
// act: agent `$0` has `.` dai
rule #Vat.dai[A] => #hashedLocation("Solidity", 5, A)

syntax Int ::= "#Vat.sin" "[" Int "]" [function]
// ---------------------------------------------
// doc: system debt assigned to `$0`
// act: agent `$0` has `.` dai
rule #Vat.sin[A] => #hashedLocation("Solidity", 6, A)

syntax Int ::= "#Vat.debt" [function]
// ---------------------------------
// doc: total dai issued from the system
// act: there is `.` dai in total
rule #Vat.debt => 7

syntax Int ::= "#Vat.vice" [function]
// ----------------------------------
// doc: total system debt
// act: there is `.` system debt
rule #Vat.vice => 8

syntax Int ::= "#Vat.Line" [function]
// ----------------------------------
// doc: global debt ceiling
// act: the global debt ceiling is `.`
rule #Vat.Line => 9

syntax Int ::= "#Vat.live" [function]
// ----------------------------------
// doc: system status
// act: the system is `. == 1 ? : not` live
rule #Vat.live => 10
```

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

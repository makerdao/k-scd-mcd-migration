Vat
```act
behaviour addui of Vat
interface add(uint256 x, int256 y) internal

stack

   #unsigned(y) : x : JMPTO : WS => JMPTO : x + y : WS

iff in range uint256

   x + y

if

   #sizeWordStack(WS) <= 1015
```

Vat slip
```act
behaviour slip of Vat
interface slip(bytes32 ilk, address usr, int256 wad)

for all

    May : uint256
    Gem : uint256

storage

    wards[CALLER_ID] |-> May
    gem[ilk][usr]    |-> Gem => Gem + wad

iff

    // act: caller is `. ? : not` authorised
    May == 1
    VCallValue == 0

iff in range uint256

    Gem + wad

calls

    Vat.addui
```

DSToken transferFrom
```act
behaviour transferFrom of DSToken
interface transferFrom(address src, address dst, uint wad)

for all
  Gem_s     : uint256
  Gem_d     : uint256
  Allowance : uint256
  Owner     : address
  Stopped   : bool

storage
  allowance[src][CALLER_ID] |-> Allowance => #if (src == CALLER_ID or Allowance == maxUInt256) #then Allowance #else Allowance - wad #fi
  balances[src] |-> Gem_s => Gem_s - wad
  balances[dst] |-> Gem_d => Gem_d + wad
  owner_stopped |-> #WordPackAddrUInt8(Owner, Stopped)

iff in range uint256
  Gem_s - wad
  Gem_d + wad

iff
  (Allowance == maxUInt256 or src == CALLER_ID) or (wad <= Allowance)
  VCallValue == 0
  Stopped == 0

if
  src =/= dst

returns 1
```
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

```act
behaviour subui of Vat
interface sub(uint256 x, int256 y) internal

stack

    #unsigned(y) : x : JMPTO : WS => JMPTO : x - y : WS

iff in range uint256

    x - y

if

    #sizeWordStack(WS) <= 1015
```

```act
behaviour mului of Vat
interface mul(uint256 x, int256 y) internal

stack

    #unsigned(y) : x : JMPTO : WS => JMPTO : #unsigned(x * y) : WS

iff in range int256

    x
    x * y

if

    // TODO: strengthen
    #sizeWordStack(WS) <= 1000
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
DSToken transfer
```act
behaviour transfer of DSToken
interface transfer(address usr, uint256 wad)

for all
  Gem_c : uint256
  Gem_u : uint256
  Owner : address
  Stopped : bool

storage
  balances[CALLER_ID] |-> Gem_c => Gem_c - wad
  balances[usr]       |-> Gem_u => Gem_u + wad
  owner_stopped |-> #WordPackAddrUInt8(Owner, Stopped)

if
  usr =/= CALLER_ID

iff in range uint256
  Gem_c - wad
  Gem_u + wad

iff
  Stopped == 0
  VCallValue == 0

returns 1
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

GemJoin
```act
behaviour join of GemJoin
interface join(address usr, uint256 wad)

for all

    Vat         : address Vat
    Ilk         : bytes32
    DSToken     : address DSToken
    May         : uint256
    Vat_bal     : uint256
    Bal_usr     : uint256
    Bal_adapter : uint256
    Owner       : address
    Stopped     : bool
    Allowed     : uint256

storage

    vat |-> Vat
    ilk |-> Ilk
    gem |-> DSToken

storage Vat

    wards[ACCT_ID]      |-> May
    gem[Ilk][usr]       |-> Vat_bal => Vat_bal + wad

storage DSToken

    allowance[CALLER_ID][ACCT_ID] |-> Allowed => #if Allowed == maxUInt256 #then Allowed #else Allowed - wad #fi
    balances[CALLER_ID] |-> Bal_usr     => Bal_usr     - wad
    balances[ACCT_ID]   |-> Bal_adapter => Bal_adapter + wad
    owner_stopped       |-> #WordPackAddrUInt8(Owner, Stopped)

iff

    VCallDepth < 1024
    VCallValue == 0
    wad <= Allowed
    Stopped == 0
    May == 1
    wad <= maxSInt256

iff in range uint256

    Vat_bal + wad
    Bal_usr     - wad
    Bal_adapter + wad

if

    CALLER_ID =/= ACCT_ID

calls

  Vat.slip
  DSToken.transferFrom
```

GemJoin exit
```act
behaviour exit of GemJoin
interface exit(address usr, uint256 wad)

for all

    Vat         : address Vat
    Ilk         : bytes32
    DSToken     : address DSToken
    May         : uint256
    Wad         : uint256
    Bal_usr     : uint256
    Bal_adapter : uint256
    Owner       : address
    Stopped     : bool

storage

    vat |-> Vat
    ilk |-> Ilk
    gem |-> DSToken

storage Vat

    wards[ACCT_ID]      |-> May
    gem[Ilk][CALLER_ID] |-> Wad => Wad - wad

storage DSToken

    balances[ACCT_ID]   |-> Bal_adapter => Bal_adapter - wad
    balances[usr]       |-> Bal_usr     => Bal_usr     + wad
    owner_stopped       |-> #WordPackAddrUInt8(Owner, Stopped)

iff

    VCallValue == 0
    VCallDepth < 1024
    Stopped == 0
    May == 1
    wad <= pow255

iff in range uint256

    Wad         - wad
    Bal_adapter - wad
    Bal_usr     + wad

if

    ACCT_ID =/= usr

calls
  Vat.slip
  DSToken.transfer
```

Vat Frob and all the stuff needed for it
```act
behaviour frob-diff-nonzero of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iv   : uint256
    Dai_w    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Can_v    : uint256
    Can_w    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    can[v][CALLER_ID] |-> Can_v
    can[w][CALLER_ID] |-> Can_w
    urns[i][u].ink    |-> Urn_ink  => Urn_ink + dink
    urns[i][u].art    |-> Urn_art  => Urn_art + dart
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art + dart
    gem[i][v]         |-> Gem_iv   => Gem_iv  - dink
    dai[w]            |-> Dai_w    => Dai_w + (Ilk_rate * dart)
    debt              |-> Debt     => Debt  + (Ilk_rate * dart)
    live              |-> Live

iff in range uint256

    Urn_ink + dink
    Gem_iv  - dink
    (Urn_ink + dink) * Ilk_spot
    (Urn_art + dart) * Ilk_rate
    (Ilk_Art + dart) * Ilk_rate
    Dai_w + (Ilk_rate * dart)
    Debt  + (Ilk_rate * dart)

iff in range int256

    Ilk_rate
    Ilk_rate * dart

iff
    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0

    (dart <= 0) or (((Ilk_Art + dart) * Ilk_rate <= Ilk_line) and ((Debt + Ilk_rate * dart) <= Line))
    (dart <= 0 and dink >= 0) or ((((Urn_art + dart) * Ilk_rate) <= ((Urn_ink + dink) * Ilk_spot)))
    (dart <= 0 and dink >= 0) or (u == CALLER_ID or Can_u == 1)

    (dink <= 0) or (v == CALLER_ID or Can_v == 1)
    (dart >= 0) or (w == CALLER_ID or Can_w == 1)

    ((Urn_art + dart) == 0) or (((Urn_art + dart) * Ilk_rate) >= Ilk_dust)

if

    u =/= v
    v =/= w
    u =/= w
    dink =/= 0
    dart =/= 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-diff-zero-dart of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iv   : uint256
    Dai_w    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Can_v    : uint256
    Can_w    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    can[v][CALLER_ID] |-> Can_v
    can[w][CALLER_ID] |-> Can_w
    urns[i][u].ink    |-> Urn_ink  => Urn_ink + dink
    urns[i][u].art    |-> Urn_art  => Urn_art
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art
    gem[i][v]         |-> Gem_iv   => Gem_iv  - dink
    dai[w]            |-> Dai_w    => Dai_w
    debt              |-> Debt     => Debt
    live              |-> Live

iff in range uint256

    Urn_ink + dink
    Gem_iv  - dink
    (Urn_ink + dink) * Ilk_spot
    Urn_art * Ilk_rate
    Ilk_Art * Ilk_rate

iff in range int256

    Ilk_rate

iff
    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0
    (dink >= 0) or (((Urn_art * Ilk_rate) <= ((Urn_ink + dink) * Ilk_spot)))
    (dink >= 0) or (u == CALLER_ID or Can_u == 1)
    (dink <= 0) or (v == CALLER_ID or Can_v == 1)
    (Urn_art == 0) or ((Urn_art * Ilk_rate) >= Ilk_dust)

if

    u =/= v
    v =/= w
    u =/= w
    dink =/= 0
    dart == 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-diff-zero-dink of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iv   : uint256
    Dai_w    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Can_v    : uint256
    Can_w    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    can[v][CALLER_ID] |-> Can_v
    can[w][CALLER_ID] |-> Can_w
    urns[i][u].ink    |-> Urn_ink  => Urn_ink
    urns[i][u].art    |-> Urn_art  => Urn_art + dart
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art + dart
    gem[i][v]         |-> Gem_iv   => Gem_iv
    dai[w]            |-> Dai_w    => Dai_w + (Ilk_rate * dart)
    debt              |-> Debt     => Debt  + (Ilk_rate * dart)
    live              |-> Live

iff in range uint256

    Urn_art + dart
    Urn_ink * Ilk_spot
    (Urn_art + dart) * Ilk_rate
    (Ilk_Art + dart) * Ilk_rate
    Dai_w + (Ilk_rate * dart)
    Debt  + (Ilk_rate * dart)

iff in range int256

    Ilk_rate
    Ilk_rate * dart

iff
    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0

    (dart <= 0) or (((Ilk_Art + dart) * Ilk_rate <= Ilk_line) and ((Debt + Ilk_rate * dart) <= Line))
    (dart <= 0) or (((Urn_art + dart) * Ilk_rate) <= (Urn_ink * Ilk_spot))
    (dart <= 0) or (u == CALLER_ID or Can_u == 1)
    (dart >= 0) or (w == CALLER_ID or Can_w == 1)
    ((Urn_art + dart) == 0) or (((Urn_art + dart) * Ilk_rate) >= Ilk_dust)

if

    u =/= v
    v =/= w
    u =/= w
    dink == 0
    dart =/= 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-diff-zero of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iv   : uint256
    Dai_w    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Can_v    : uint256
    Can_w    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    can[v][CALLER_ID] |-> Can_v
    can[w][CALLER_ID] |-> Can_w
    urns[i][u].ink    |-> Urn_ink  => Urn_ink
    urns[i][u].art    |-> Urn_art  => Urn_art
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art
    gem[i][v]         |-> Gem_iv   => Gem_iv
    dai[w]            |-> Dai_w    => Dai_w
    debt              |-> Debt     => Debt
    live              |-> Live

iff in range uint256

    Urn_ink * Ilk_spot
    Urn_art * Ilk_rate
    Ilk_Art * Ilk_rate

iff in range int256

    Ilk_rate

iff
    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0
    (Urn_art == 0) or ((Urn_art * Ilk_rate) >= Ilk_dust)

if

    u =/= v
    v =/= w
    u =/= w
    dink == 0
    dart == 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-same-nonzero of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iu   : uint256
    Dai_u    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    urns[i][u].ink    |-> Urn_ink  => Urn_ink + dink
    urns[i][u].art    |-> Urn_art  => Urn_art + dart
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art + dart
    gem[i][u]         |-> Gem_iu   => Gem_iu  - dink
    dai[u]            |-> Dai_u    => Dai_u + (Ilk_rate * dart)
    debt              |-> Debt     => Debt  + (Ilk_rate * dart)
    live              |-> Live

iff in range uint256

    Urn_ink + dink
    Gem_iu  - dink
    (Urn_ink + dink) * Ilk_spot
    (Urn_art + dart) * Ilk_rate
    (Ilk_Art + dart) * Ilk_rate
    Dai_u + (Ilk_rate * dart)
    Debt  + (Ilk_rate * dart)

iff in range int256

    Ilk_rate
    Ilk_rate * dart

iff

    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0
    (dart <= 0) or (((Ilk_Art + dart) * Ilk_rate <= Ilk_line) and ((Debt + Ilk_rate * dart) <= Line))
    (dart <= 0 and dink >= 0) or ((((Urn_art + dart) * Ilk_rate) <= ((Urn_ink + dink) * Ilk_spot)))
    (u == CALLER_ID or Can_u == 1)
    ((Urn_art + dart) == 0) or (((Urn_art + dart) * Ilk_rate) >= Ilk_dust)

if

    u == v
    v == w
    u == w
    dink =/= 0
    dart =/= 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-same-zero-dart of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iu   : uint256
    Dai_u    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    urns[i][u].ink    |-> Urn_ink  => Urn_ink + dink
    urns[i][u].art    |-> Urn_art  => Urn_art
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art
    gem[i][u]         |-> Gem_iu   => Gem_iu  - dink
    dai[u]            |-> Dai_u    => Dai_u
    debt              |-> Debt     => Debt
    live              |-> Live

iff in range uint256

    Urn_ink + dink
    Gem_iu  - dink
    (Urn_ink + dink) * Ilk_spot
    Urn_art * Ilk_rate
    Ilk_Art * Ilk_rate

iff in range int256

    Ilk_rate

iff

    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0
    (dink >= 0) or (((Urn_art * Ilk_rate) <= ((Urn_ink + dink) * Ilk_spot)))
    (u == CALLER_ID or Can_u == 1)
    (Urn_art == 0) or ((Urn_art * Ilk_rate) >= Ilk_dust)

if

    u == v
    v == w
    u == w
    dink =/= 0
    dart == 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-same-zero-dink of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iu   : uint256
    Dai_u    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    urns[i][u].ink    |-> Urn_ink  => Urn_ink
    urns[i][u].art    |-> Urn_art  => Urn_art + dart
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art + dart
    gem[i][u]         |-> Gem_iu   => Gem_iu
    dai[u]            |-> Dai_u    => Dai_u + (Ilk_rate * dart)
    debt              |-> Debt     => Debt  + (Ilk_rate * dart)
    live              |-> Live

iff in range uint256

    Urn_ink * Ilk_spot
    (Urn_art + dart) * Ilk_rate
    (Ilk_Art + dart) * Ilk_rate
    Dai_u + (Ilk_rate * dart)
    Debt  + (Ilk_rate * dart)

iff in range int256

    Ilk_rate
    Ilk_rate * dart

iff

    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0
    (dart <= 0) or (((Ilk_Art + dart) * Ilk_rate <= Ilk_line) and ((Debt + Ilk_rate * dart) <= Line))
    (dart <= 0) or (((Urn_art + dart) * Ilk_rate) <= (Urn_ink * Ilk_spot))
    (u == CALLER_ID or Can_u == 1)
    ((Urn_art + dart) == 0) or (((Urn_art + dart) * Ilk_rate) >= Ilk_dust)

if

    u == v
    v == w
    u == w
    dink == 0
    dart =/= 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```

```act
behaviour frob-same-zero of Vat
interface frob(bytes32 i, address u, address v, address w, int dink, int dart)

for all

    Ilk_rate : uint256
    Ilk_line : uint256
    Ilk_spot : uint256
    Ilk_dust : uint256
    Ilk_Art  : uint256
    Urn_ink  : uint256
    Urn_art  : uint256
    Gem_iu   : uint256
    Dai_u    : uint256
    Debt     : uint256
    Line     : uint256
    Can_u    : uint256
    Live     : uint256

storage

    ilks[i].rate      |-> Ilk_rate
    ilks[i].line      |-> Ilk_line
    ilks[i].spot      |-> Ilk_spot
    ilks[i].dust      |-> Ilk_dust
    Line              |-> Line
    can[u][CALLER_ID] |-> Can_u
    urns[i][u].ink    |-> Urn_ink  => Urn_ink
    urns[i][u].art    |-> Urn_art  => Urn_art
    ilks[i].Art       |-> Ilk_Art  => Ilk_Art
    gem[i][u]         |-> Gem_iu   => Gem_iu
    dai[u]            |-> Dai_u    => Dai_u
    debt              |-> Debt     => Debt
    live              |-> Live

iff in range uint256

    Urn_ink * Ilk_spot
    Urn_art * Ilk_rate
    Ilk_Art * Ilk_rate

iff in range int256

    Ilk_rate

iff

    VCallValue == 0
    Live == 1
    Ilk_rate =/= 0
    (Urn_art == 0) or ((Urn_art * Ilk_rate) >= Ilk_dust)

if

    u == v
    v == w
    u == w
    dink == 0
    dart == 0

calls

    Vat.addui
    Vat.subui
    Vat.mului
    Vat.muluu
```
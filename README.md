# One-way Cross-chain Permissionless Router Bridge

So.

I've said I would have added PayPal USD (PYUSD) token support for purchasing
applications on the
[Ur uncensorable Life and DogeOS user repository and application store](
  https://github.com/themartiancompany/ur)
in the context of the
["seamless transactions, infinite possibilities hackaton."](
  https://github.com/themartiancompany/seamless-transactions-infinite-possibilities-hackaton)

Still Google RPC supports only Ethereum mainnet and Holesky,
and doing stuff on there is stupidly expensive, so if I simply
deployed an Ethereum Ur instance still probably nobody
would ever use it to like buy $1 applications, which actually
it seems to be the biggest revenue pool for those, and only
if you sell millions.

So more or less an Ethereum Ur would be a fun trick
but it would be completely unusable in practice.

Now I can still add support for PYUSD on Gnosis,
still I wouldn't be using the Google RPC at all.

Also probably PayPal wouldn't like much a project
which roughly exposes its token's customers to
a stable-coin baked network from a competitor.

Not that I care much frankly, as what the Ur
is supposed to do is to make people able to buy
uncensorable applications whichever way,
so I will probably do it just the same.

Anyway, I was wondering if for the hackaton
the right compromise might actually be
to set up a one-way cross-chain bridge
with a permissionless router, which actually
on Gnosis wouldn't be one-way as there is
the official centralized one available
I could use to move the money back.

So I've never written a bridge nor I've ever read
the code for one, still from what I've come
to understand none of the existing
works with a permissionless router, or at least
they all depend on usually a single
actor keeping a server up.

Since it's happening nowadays that I'm writing
a lot of stuff which is making lots
of centralized services virtually useless
without getting a dime nor any kind of
recognizement from this activity, I've thought
"why not doing the same also for network
bridges? in the end its not like anybody will
ever thank, provide for or support me
whatever I can do, so why not proving once
again humanity the best thing it could happen
to earth is for it to disappear?".

Premise I have not thought in the slightest about the
caveats of the following scheme, that I keep
forgetting how a 4-way handshake works and
that I have a terrible headache right now,
I was imagining something like the following.

If anybody sees any conceptual or practical issue
with it, if you tell me I could probably think
you're not like unworthy of the oxygen you're breathing.

### The scheme

Feel free to replace `XDAI` with the symbol for the
centralized bridged version of `PYUSD`.

#### Initial data

```
  Balance:
    user A:
      +10 PYUSD
      +10 XDAI
    user B:
      +10 PYUSD
        0 GPYUSD
        0 XDAI
    vault E:
        0 PYUSD
    vault G:
        0 XDAI
        0 GPYUSD
```

1) User A deposits 10 XDAI on Gnosis vault and 10 PYUSD on Ethereum vault.

```
  Transaction E1:
    user A        =>     vault E
    -10PYUSD             +10 PYUSD
  
  Transaction G1:
    user A        =>     vault G
    -10 XDAI             +10 XDAI
                         +10 GPYUSD

  Balance:
    user A:
        0 PYUSD
        0 XDAI
    user B:
      +10 PYUSD
        0 GPYUSD
        0 XDAI
    vault E:
      +10 PYUSD
    vault G:
      +10 XDAI
      +10 GPYUSD
```

2) User B requests to bridge 10 PYUSD to Ethereum vault from user A with timeout `t` and locks
   user A funds on Gnosis vault by presenting the signature of the request.

```
  Transaction E2:
    user B        =>     vault E
    -10 PYUSD   (A, t)   +10 PYUSD

  Transaction G2:
    user B        =>     vault G
                sig(E2)

  Balance:
    user A:
        0 PYUSD
        0 XDAI
    user B:
        0 PYUSD
        0 GPYUSD
        0 XDAI
    vault E:
      +20 PYUSD
    vault G:
      +10 XDAI
      +10 GPYUSD
```
 
3) User A presents valid signature to Gnosis vault of user B request to bridge 10 PYUSD on Ethereum vault,
   so user B is allowed to receive 8 GPYUSD from Gnosis vault before timeout is reached.

```
  Transaction G3:
    user A        =>     vault G
                sig(E2)
```

4) User B sends request to receive 8 GPYUSD from Gnosis vault and user A receives 1 XDAI.

```
  Transaction G4:
    user B        =>     vault G 
     +8 GPYUSD            -8 GPYUSD
    vault G       =>     user A
     -1 XDAI              +1 XDAI

  Balance:
    user A:
        0 PYUSD
       +1 XDAI
    user B:
        0 PYUSD
       +8 GPYUSD
        0 XDAI
    vault E:
      +20 PYUSD
    vault G:
       +9 XDAI
       +2 GPYUSD
```

5) Before timeout `t` is reached user A presents valid signature to Ethereum vault of user B receiving the
   8 GPYUSD so he is allowed to take user B 10 PYUSD and his own PYUSD.

```
  Transaction E3:
    user A        =>     vault E
                sig(G4)
    vault E       =>     user A
    -20 PYUSD            +20PYUSD

  Balance:
    user A:
      +20 PYUSD
       +1 XDAI
    user B:
        0 PYUSD
       +8 GPYUSD
        0 XDAI
    vault E:
        0 PYUSD
    vault G:
       +9 XDAI
       +1 GPYUSD
```

6) At a later time user B or somebody else returns the GPYUSD to the vault G and redeems the XDAI.

```
  Transaction G5:
    user B        =>     vault G
     -8 GPYUSD           +8 GPYUSD
    vault G       =>     user B
     -7 XDAI             +7 XDAI
    vault G       =>     null address
     -7 GPYUSD    =>     +7 GPYUSD

  Balance:
    user A:
      +20 PYUSD
       +1 XDAI
    user B:
        0 PYUSD
        0 GPYUSD
       +7 XDAI
    vault E:
        0 PYUSD
    vault G:
       +2 XDAI
       +1 GPYUSD
```

7) The bridge author takes some money to further think about some
   other way to make the world a better place.

```
  Transaction G6:
    bridge author =>     vault G
                dinner
    vault G       =>     bridge author
     -1 XDAI             +1 XDAI
    vault G       =>     null address
     -1 GPYUSD    =>     +1 GPYUSD

  Balance:
    bridge author:
       +1 XDAI
    vault E:
        0 PYUSD
    vault G:
       +1 XDAI
       +1 GPYUSD
```

#### Observations

- User A can be anybody, so user B and user A can be the same person.
- The bridge retains a share of the funds.
- In case user A and user B are the same person and the Ethereum transactions
  are never actually transmitted to the network, the GPYUSD token retains
  its value.
- In case user A and bridge author are the same person, the bridge
  share must be large enough to cover the expenses for network fees on
  Ethereum and Gnosis network.
- Why I've never heard about
  [cross-chain clocks](
    https://ethereum.stackexchange.com/questions/168462/has-anybody-published-code-for-a-cross-chain-clock)?

### License

The content in this page, as the rest of this repository is released under the terms of the GNU Affero General Public License version 3.

If you're wondering why I do release prose with a code license, the answer may be I am a computer.

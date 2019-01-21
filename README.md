# fdb-erlang

Erlang bindings for [FoundationDB](https://foundationdb.com/).

This is still very early alpha.


## Prerequisites

Developed and tested on ubuntu only (for now).
It requires you to have Erlang and [the FoundationDB client libraries installed](https://foundationdb.com/documentation/api-general.html#installing-client-binaries).

Due to the way `enif_send` is implemented in Erlang `NIF`s, your system needs to run with at least 2 core's.

#### Update
Apparently you can also work around this by using `erl -smp enable`.

Big thanks to [Valery Meleshkin](https://twitter.com/sum3rman) for the tip!

## Compilation and testing

```bash
git clone git@github.com:HappyPancake/fdb-erlang.git
cd fdb-erlang
chmod u+x rebar
./rebar get-deps compile eunit -v

(venv) jiangphs-mbp-2:fdb-erlang jiangph$  ./rebar get-deps compile eunit -v
==> fdb-erlang (get-deps)
==> fdb-erlang (compile)
==> fdb-erlang (eunit)
======================== EUnit ========================
module 'directory'
module 'maybe'
module 'gen_fdb'
module 'subspace'
subspace_test: basic_test (module 'subspace_test')...[0.102 s] ok
module 'tuple'
module 'tuple_test'
  tuple_test: pack_test...ok
  tuple_test: pack_order_test...ok
  tuple_test: unpack_test...ok
  [done in 0.009 s]
module 'fdb'
module 'fdb_nif'
module 'stress_test'
module 'directory_test'
module 'fdb_test'
  fdb_test: hello_world_test...[0.038 s] ok
  fdb_test: transaction_test...[0.045 s] ok
  fdb_test: range_test...[0.017 s] ok
  [done in 0.109 s]
fdb_nif_test: store_and_retrieve_test (module 'fdb_nif_test')...[0.001 s] ok
=======================================================
  All 8 tests passed.
```

## Example usage

This is the simplest possible usage:
```erlang
tojans@ubuntu:/tmp/fdb-erlang$ erl -pa ebin
Erlang R16B02 (erts-5.10.3) [source-5d89ddd] [64-bit] [async-threads:10] [kernel-poll:false]

Eshell V5.10.3  (abort with ^G)
1> fdb:init().
ok
2> fdb:api_version(100).
0
3> DB = fdb:open().
{db,<<>>}
4> fdb:get(DB,<<"Hello">>).
not_found
5> fdb:set(DB,<<"Hello">>,<<"World">>).
ok
6> fdb:get(DB,<<"Hello">>).            
<<"World">>
7> fdb:get(DB,<<"Hello2">>,some_default_value).
some_default_value
8> fdb:clear(DB,<<"Hello">>).
ok
9> fdb:get(DB,<<"Hello">>).
not_found
10>
```

If you want to have a transaction, you need to invoke code like this:
```erlang
10> fdb:transact(DB, fun(Tx) ->                         
10>   fdb:set(Tx,<<"Hello">>,<<"World">>),            
10>   fdb:set(Tx,<<"xyz">>,<<"abc">>),                
10>   [fdb:get(Tx,<<"xyz">>)|fdb:get(Tx,<<"Hello">>)]   
10> end).
[<<"abc">>|<<"World">>]
11>
```
It will invoke the lambda multiple times until the whole action succeeds or it seems unreasonable to try any longer.
If the code in the lambda crashes, the transaction will be rolled back.

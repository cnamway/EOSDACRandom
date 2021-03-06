# EOSDACRandom
A random number generator smart contract.

## Attention
```
EOSDACRandom is related to oracleserver, an oracle market contract implemented by oraclechain, which
currently is not open-sourced. Therefore building this project would be failed, the suggestion is remove
all oracleserver related code.
```

### participator
1. random contract (aka contract), an oracle instance.
2. data feeder
3. consumer

### procedure
#### contract
  * set seed size, e.g n.

#### data feeders
  * all n data feeders send hash of seed, e.g h1.
  * all data feeders send seed, e.g s, (MUST wait for h1 sent, otherwise this will fail).
  * calculate hash of seed s, get h2, compare h1 and h2, if equals, we put seed s in seed pool, otherwise, put provider in blacklist.

#### consumers
  * first need implment a `getrandom` interface.
  * want a random number, register himself to random contract.
  * random contract generator a random number using seed pool. After it's done, clear pool, wait for next round.
  * send consumer this random number using `getrandom` by `SEND_INLINE_ACTION`. After it's done, clear geters table, wait for next round.

----

### test

#### How to test EOSDACRandom contract?

```
1. deploy EOSDACToken contract
2. create OCT
3. issue OCT
4. create account
  4.1 eosdacrandom
  4.2 requester
  4.3 seeder1, seeder2, seeder3
5. transfer OCT to requester, seeder1, seeder2, seeder3
6. deploy EOSDACRandom contract
7. deploy requester contract
8. push action eosdacrandom setsize '[3]' -p eosdacrandom
9. push action eosdacrandom regrequest '["requester", 1]' -p requester
10. push action eosdacrandom sendhash '["seeder1", "A665A45920422F9D417E4867EFDC4FB8A04A1F3FFF1FA07E998E86F7F7A27AE3", "OCT"]' -p seeder1
11. push action eosdacrandom sendhash '["seeder2", "A665A45920422F9D417E4867EFDC4FB8A04A1F3FFF1FA07E998E86F7F7A27AE3", "OCT"]' -p seeder2
12. push action eosdacrandom sendhash '["seeder3", "A665A45920422F9D417E4867EFDC4FB8A04A1F3FFF1FA07E998E86F7F7A27AE3", "OCT"]' -p seeder3
13. push action eosdacrandom seedseed '["seeder1", 1, "OCT"]' -p seeder1
14. push action eosdacrandom sendseed '["seeder2", 1, "OCT"]' -p seeder2
15. push action eosdacrandom sendseed '["seeder3", 1, "OCT"]' -p seeder3
16. here requester should get the random number.
```

#### example

```
$ cleos system newaccount eosio octtothemoon EOS6RLatEKdW3LGZ4Kfftf7xAEmqVkkFjsjG4uEefuatSZyDLk261 EOS6RLatEKdW3LGZ4Kfftf7xAEmqVkkFjsjG4uEefuatSZyDLk261 --stake-cpu "1.0000 SYS" --stake-net "1.0000 SYS" --buy-ram "1.0000 SYS"

$ cleos system newaccount eosio eosdacrandom EOS62wJ1teiN23sdB2ynQE2za715FRwuXC7XN5xa1hu16QLs8EkV9 EOS62wJ1teiN23sdB2ynQE2za715FRwuXC7XN5xa1hu16QLs8EkV9 --stake-cpu "1.0000 SYS" --stake-net "1.0000 SYS" --buy-ram "1.0000 SYS"

cleos set account permission eosdacrandom active '{"threshold": 1,"keys": [{"key": "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","weight": 1}],"accounts": [{"permission":{"actor":"eosdacrandom","permission":"eosio.code"},"weight":1}]}' owner -p eosdacrandom

cleos create account eosio requester EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

cleos create account eosio seeder1 EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

cleos create account eosio seeder2 EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

cleos create account eosio seeder3 EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

$ cleos set contract octtothemoon ./ eosdactoken.wasm eosdactoken.abi

$ cleos set contract eosdacrandom ./ eosdacrandom.wasm eosdacrandom.abi

$ cleos push action octtothemoon create '["octtothemoon", "100000000.0000 OCT"]' -p octtothemoon

$ cleos push action octtothemoon issue '["octtothemoon", "100000000.0000 OCT"]' -p octtothemoon

$ cleos push action oracleserver registerorc '["eosdacrandom", "random", 10, 10]' -p oracleserver

cleos push action eosdactoken transfer '["eosdactoken", "eosdacrandom", "1000.0000 OCT", ""]' -p eosdactoken

cleos push action eosdactoken transfer '["eosdactoken", "requester", "1.0000 OCT", ""]' -p eosdactoken

cleos push action eosdactoken transfer '["eosdactoken", "seeder1", "1.0000 OCT", ""]' -p eosdactoken

cleos push action eosdactoken transfer '["eosdactoken", "seeder2", "1.0000 OCT", ""]' -p eosdactoken

cleos push action eosdactoken transfer '["eosdactoken", "seeder3", "1.0000 OCT", ""]' -p eosdactoken


cleos set contract eosdacrandom ./eosdacrandom -p eosdacrandom

cleos set contract requester ./requester -p requester

cleos push action eosdacrandom setsize '[1]' -p eosdacrandom
cleos push action eosdacrandom setsize '[1]' -p eosdacrandom

cleos push action eosdacrandom regrequest '["requester", 1]' -p requester

cleos push action eosdacrandom sendhash '["seeder1", "6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b", "OCT"]' -p seeder1

cleos push action eosdacrandom sendseed '["seeder1", 1, "OCT"]' -p seeder1
```

# yokozuna_perf_patch
Set of Yokozuna changes to improve bulk load performance.

## CentOS + Riak 2.0.4 Installation

#### Download and Configure

```
git clone https://github.com/basho-labs/yokozuna_perf_patch.git
cd yokozuna_perf_patch
PATH=$PATH:/usr/lib64/riak/erts-5.10.3/bin
export PATH
```

#### Compile Patch and Move to `basho-patches`

```
mkdir -p ebin
erlc -pa src/ -I include/ -o ebin/ src/yz_solr.erl
cp ebin/*.beam /usr/lib64/riak/lib/basho-patches/
```

## List of changes

#### `yz_solr.erl`

Removed the delete by query so it is not executed on every new document index. The delete by query specifically removes siblings from solr before inserting. If siblings are not used (in the case of using CRDTs, siblings are not exposed to Solr), this delete by query is unecessary.

Changed:

```
index(Core, Docs, DelOps) ->
    Ops = {struct,
           [{delete, encode_delete(Op)} || Op <- DelOps] ++
               [{add, encode_doc(D)} || D <- Docs]},
```

To: 

```
index(Core, Docs, _DelOps) ->
    %% BEGIN YZ_PERF_PATCH CODE (removed sibling delete, replace if using siblings)
    Ops = {struct,
           [{add, encode_doc(D)} || D <- Docs]},
    %% END YZ_PERF_PATCH CODE
```
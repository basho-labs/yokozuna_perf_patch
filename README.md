# yokozuna_perf_patch
Set of Yokozuna changes to improve bulk load performance.

## Patch Riak

```
cp ebin/*.beam /usr/lib64/riak/lib/basho-patches/
```

### For testing, compilation of this project can be done like so

```
mkdir -p ebin
erlc -pa src/ -I include/ -o ebin/ src/*.erl
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
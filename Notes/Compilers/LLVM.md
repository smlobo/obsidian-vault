# `opt` options
* call graph
```
opt -passes=dot-callgraph -callgraph-show-weights -callgraph-heat-colors -disable-output loop.ll
```
![[loop.ll.callgraph.pdf]]
* CFG
```
opt -passes=dot-cfg -cfg-func-name=main -cfg-dot-filename-prefix=cfg -cfg-weights -cfg-heat-colors -disable-output loop.ll
```
![[cfg.loop.main.pdf]]
* cfg, dominator tree, & (loop) ddg
```
opt -passes='verify,function(dot-cfg,dot-dom,loop(dot-ddg))' \
    -cfg-dot-filename-prefix=cfg \
    -cfg-heat-colors=false \
    -disable-output example.ll
```
![[cfg.svg]]
![[dominators.svg]]
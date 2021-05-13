---
title: "makeに引数が正しく渡されているか確認する"
emoji: "🐰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["make", "makefile"]
published: true
---

# よくある引数確認方法と問題点

Makefileで引数が確認されてるか確認する方法として擬似ターゲット（`.PHONY`）と`ifdef`や`ifeq`を組み合わせる方法がよく紹介されています。

```makefile:Makefile
PARAM :=

.PHONY: main
main: base-program
	some-command

.PHONY: base-program
base-program: __require_PARAM
	some-command ${PARAM}

.PHONY: __require_PARAM
__require_PARAM:
ifndef PARAM
	$(error PARAM is not defined; you must specify PARAM like $$ make PARAM=xxx task)
endif
```

```shell
$ make PARAM=xxx main
```

makeをタスクランナーとして使う場合はこれでも良いですが、ビルドツールとして使うときは依存ターゲットに擬似ターゲットが含まれているとビルドキャッシュが効かなくなるのでとても不便です（特にビルド時間が長いとき）。

# 解決法

`define`とbashの組み合わせで解決できます。
この場合、擬似ターゲットを使わないので、ビルドキャッシュを利用することができます。

```makefile:Makefile
PARAM :=

.PHONY: main
main: base-program
	some-command

.PHONY: base-program
base-program: 
	$(call __require_PARAM)
	some-command ${PARAM}

define __require_PARAM
    @bash -c "if [ '${PARAM}' = '' ]; then echo 'PARAM is not defined; you must specify PARAM like $$ make PARAM=xxx task'; exit 1; fi"
endef
```


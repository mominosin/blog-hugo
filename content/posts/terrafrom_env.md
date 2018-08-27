---
title: "Terrafromで環境変数の値を利用する"
date: 2018-08-27T23:34:28+09:00
draft: false
categories: ["terraform","aws"]
tags: ["tfenv"]
---

# はじめに
久々にブログを書き始めようと思い、一から作成しなおしてみました。
TwitterなどSNSなどでは `もみあげまん` だったり `もみん` だったりいろいろな呼ばれ方をしています。

主にインフラをみており、AWS、Terraform等の利用が多いのでそのあたりをアウトプットする場として利用しようと思っています。

ではそろそろ本題

# terraformで環境変数の値を利用する
とりあえずなにからしようかなと思ったりはしたんですが、
AWSを利用してTerraformを利用とすると考えた場合、
アクセスキーのソースコード直書きはいかかがなものかというものがふと思い浮かんがので最初にそのことを書いてみようと思います。

## 環境変数を設定する
terraformで利用する環境変数は、とりあえず設定してあるものすべてが使えるわけではなく `TF_VAR_` というprefixのついた環境変数だけがterraformで利用できます。  
ただ `TF_VAR` のついた環境変数を準備しとけば利用できるのかと言われればそうではなく、pfrefixを取り除いた名前でTerraform側で定義しておく必要があります。  
例えば、`TF_VAR_hoge`,`TF_VAR_huga` をterraformで利用したい場合は下記のような設定を作って置く必要があります。

```tf:variable.tf
variable "hoge" {}
variable "huga" {}
```

variableはTerraform内で利用できる変数のようなものです、`{}` 内に値も定義できるのですが、外部から受け取る前提のものは空にしておくのが良いかと思います。  
試しにこのような内容のファイル用意します。  
そして環境変数設定後に `terraform console` で動作確認します。

```sh
# 上記で言ったようにprefix付きの環境変数を準備
$ export TF_VAR_hoge=hogefuga 
# 試しにprefixなしの環境変数も準備
$ export hoge=fizzbuzz
# terraform consoleを実行
$ terraform console
> var.hoge        ## TF_VAR_hoge の値が取得できている
hogefuga
> var.huga        ## 環境変数の設定されていない値はエラーになる
1:3: unknown variable accessed: var.huga in:
```
※ Terraform Console は変数や、stateの値やTerraform関数の確認に使えたりするのですが、詳細はまたいつか

このようにして、Terraformに渡せる環境変数を設定します。 
本家のドキュメントはこのあたりみてもらったらいいのかなと思います。
https://www.terraform.io/docs/configuration/environment-variables.html#tf_var_name


## AWSのアクセスキーを環境変数にするには
とりあえずproviderのの設定を書いたファイルを用意しました。

```terraform
variable "access_key" {}
variable "secret_key" {}

provider "aws" {
  region     = "ap-northeast-1"
  access_key = "${var.access_key}"
  secret_key = "${var.secret_key}"
}
```

regionは普通はそうそう切り替えないだろうと思ったので直打ちしときました…。
アクセスキーとシークレットキーは直書きしちゃいけないのでそこだけは環境変数でやりましょう。

使い方は同じで、 `TF_VAR_access_key` `TF_VAR_secret_key` の環境変数を設定してもらい、
Terraformのコマンドを実行してもらえばいいだけです。

## 実は環境変数していしなくてもいい
先程の項のファイルを利用して、環境変数を利用せずTerraformを実行してみます。

```
$ terraform plan
var.access_key
  Enter a value:
```

すると未定義の変数を入れろといわれるので、そこに環境変数に設定すべき値をいれてもらったら、普通に実行できます。

ただ、開発中でなんども `terraform plan` 等するなら環境変数設定しておくほうが楽だと思います。

以上で、Terraformで環境変数を利用する方法のご紹介でした。  
今回は例としてAWSのシークレットキーの話をしましたがもちろんこれはGCPでもAzureでも同じことなので、読み替えて利用していただけたらと思います。

# どうでもいい話
```sh
$ export TF_VAR_hoge=fuga
```

環境変数の設定でこうやりますが。

```sh
$ TF_VAR_hoge=fuga
```

`export` 無しだとどうなるかと言いますと、これは `シェル変数` となってそのシェル内でしか反映されません。

なのでコマンド実行して子プロセスになってしまうとその変数は引き継がれないので、Terraformでは読み込まれませんのでご注意ください。

ちゃんと `export` を使って環境変数の設定をしてください。

おわり

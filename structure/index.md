プログラムの基本動作
=

構成要素
-

プログラムを構成するための要素は少ないです。

- 変数
- 分岐
- ループ
- サブルーチン
- 例外

基本はこれらを組み合わせて目的の処理を作って行くだけの作業です。
プログラミングパラダイムによって表現方法が異なる部分があるので、そこは慣れていく必要があります。
ループ処理やサブルーチンは差が大きいところかもしれません。

ループ
-

リストの値を合計する処理を考えます。
（これらのサンプルはJavascriptなのでブラウザで試すことが出来ます）

ぱっと思いつくのはリストの要素ぶんループして、足し算を繰り返す方法です。
伝統的な？ループ処理`for`で決めた回数同じ処理を回して、結果を得ます。

```js
const list = [1,2,3,4];
let total = 0;
for(let i=0; i<list.length; i++){
    total += list[i];
}
console.log(total);//=>10
```

オブジェクト指向的に考えると、リストオブジェクトにはリスト操作のメソッドが準備されている＝ループ処理も準備されているので、それを使うことを考えます。
Javascriptだと、Array.forEachがそれにあたります。

```js
const list = [1,2,3,4];
let total = 0;
list.forEach((e)=>{
    total += e;
});
console.log(total);//=>10
```

もう少し調べてみると、Array.reduceというループ用メソッドがあることが分かります。
リストループの際に前の計算結果を持ちまわって結果を出す、まさに今回の用途にピッタリです。

```js
const list = [1,2,3,4];
const total = list.reduce((acc, crr)=>{
    return acc + crr;
}, 0);
console.log(total);//=>10
```

合計算出のプログラムを3つ例示しました。
これらの差は何でしょう？良し悪しは何に基づいて判断すればよいでしょう？

コードの良し悪しは「読みやすさ、短さ、変数の少なさ」が基準になりますが、読みやすさを損なってまでコードを短くしたり、変数を減らす価値はない、という点には注意が必要です。


コーディング
-

上級者は最初から分かりやすく美しいコードが書ける、というのは幻想です。
誰もが**推敲**しながら仕上げてます。
テキストデータをオブジェクトのリストに変換する処理を例に、コードを書いてみましょう。

### 要求

以下のテキストデータを、オブジェクトのリストに変換する。

```
name,age,birth,sex,blood
shimizu,29,1992-5-6,F,O
akamatsu,30,1991-6-5,M,B
miyai,23,1999-3-9,M,AB
tanno,38,1984-4-19,M,O
kawamura,24,1998-4-21,F,AB
watanabe,21,2001-1-23,F,AB
sato,33,1989-4-15,F,O
taga,35,1987-4-17,M,A
mizuno,23,1998-7-11,M,A
nakai,31,1990-7-28,F,B
```

### ひな型

Chromeのデバッグツールだけではしんどいコードなので、以下のひな型（`index.html`）を元にコーディングを進めます。

```html
<script>

const input_data = `
    name,    age,birth,    sex,blood
    shimizu, 29, 1992-5-6, F,  O
    akamatsu,30, 1991-6-5, M,  B
    miyai,   23, 1999-3-9, M,  AB
    tanno,   38, 1984-4-19,M,  O
    kawamura,24, 1998-4-21,F,  AB
    watanabe,21, 2001-1-23,F,  AB
    sato,    33, 1989-4-15,F,  O
    taga,    35, 1987-4-17,M,  A
    mizuno,  23, 1998-7-11,M,  A
    nakai,   31, 1990-7-28,F,  B
`;

<!--
    以降のコードはここに記述します
-->

</script>
```

### ステップ１：手順の通り書き下す

まずは処理の流れを頭の中で考えた通りに書くのが良いと思います。

```js
//結果格納用
const objects = [];
//入力データを行分割
const lines = input_data.split(/[\r\n]+/);

//行ごとにループ
for(let i=0; i<lines.length; i++){
    //行を整形（前後の空白を除去）
    const line = lines[i].replace(/^\s+|\s+$/, "");
    if(!line){ continue; }//空行はパス

    //列に分割
    const colmuns = line.split(/\s*,\s*/);
    const [name, age, birth, sex, blood] = [...colmuns];
    if(name == "氏名"){ continue; }//見出し行はパス

    //結果作成＆格納
    const object = {
        name: name,
        age: age,
        birth: birth,
        sex: sex,
        blood: blood,
    };
    objects.push(object);
}
//結果出力
console.log(objects);
```

### ステップ２：部分でサブルーチン化

ステップ１のプログラムはよく言えば素直ですが、全部読まないと要求が読み取りにくいです。
適切にサブルーチン化すると、全てを読まなくても大意がつかめるようになります。

```js
//行→オブジェクト変換サブルーチン
const line_to_object = function(line){
    const [name, age, birth, sex, blood] = line.split(/\s*,\s*/);
    return { name, age, birth, sex, blood };
};

//入力行の整形
const lines = input_data
    .split(/[\r\n]+/)//改行で分割
    .map((line)=>{ return line.replace(/^\s+|\s+$/, "");})//前後の空白を除去
    .filter((line)=>{ return !!line; })//空行を除去
    .slice(1)//見出し行を除去

//変換処理
const objects = lines.map(line_to_object);//行→オブジェクト変換

//結果出力
console.log(objects);
```

入力行の整形と変換処理も1つにまとめてしまうこともできますが、入力データの整形は前処理、変換処理は具体的な目的なので、混ぜてしまうとコードが読みにくくなるので分けました。

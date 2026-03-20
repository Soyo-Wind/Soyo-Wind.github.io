# Flags

このプログラミング言語は、Flag形式のスクリプトファイル(.flg)を読み取り、状態（フラグ）に応じてアクションを実行する、C# 製のインタプリタです。ステートマシン的な動作が可能。

---

## BNF

~c
<program>        ::= <line> { <line> }

<line>           ::= <flag> ',' <actions> ';'
                  | <flag> ';'

<flag>           ::= <identifier>

<actions>        ::= <action> { ';' <action> }

<action>         ::= '!' <identifier>              ; 削除
                  | '$' <output_value>             ; 出力
                  | '_'<identifier>':'<qur>               ; メモリに追加
                  | '=' <identifier> '\' <string>  ; 値代入
                  | <identifier>                   ; 通常フラグ実行（トリガー）

<output_value>   ::= '"' <string>                 ; 文字列リテラル
                  | '~' <identifier>               ; メモリの値

<qur> ::= <string> | <int>

<identifier>     ::= <letter> { <letter_or_digit> }

<letter>         ::= 'a' | 'b' | ... | 'z' | 'A' | ... | 'Z' |
<digit>          ::= '0' | '1' | ... | '9'
<letter_or_digit> ::= <letter> | <digit>

<string>         ::= { <any_char_except_backslash_or_quote> }
<int>            ::= {<digit>}
~e

---

## 主な機能

- テキストファイルからスクリプトを読み込み実行
- 条件に応じたアクションの分岐処理
- メモリ（フラグと値）による状態保持
- デバッグモードによる実行トレース表示

## スクリプト形式

スクリプトは1行ごとに フラグ名 と アクション列 (プレフィックス+値) を定義します。

**書式**

~c
フラグ名, アクション1; アクション2; アクション3;
~e

**Hello World**

~a
adam, _start;
start, $Hello world; _end;
~e

このスクリプトの流れ：

初期フラグ adam がメモリにあるため処理開始

start フラグが追加される

hello world が表示される

end フラグが追加され、終了

**対応アクション一覧**

|プレフィックス|動作内容|
|-------|-----------------|
|\_フラグ名:str|指定フラグ(str型)をメモリに追加|
|\_フラグ名:int|指定フラグ(int型)をメモリに追加|
|!フラグ名|指定フラグをメモリから削除|
|$"文字列|文字列リテラルを標準出力に表示|
|$\~フラグ名|指定フラグの Value を出力|
|=フラグ\値|指定のフラグに値を代入|
|（その他）|未知のアクションとして例外をスロー|

**メモリ構造**

Flag は Name（フラグ名）と Value（オプションの値）を持つ構造体またはクラスです。

実行開始時点で adam フラグが自動でメモリに追加されます。

end フラグがメモリに存在すると、ループを抜けてプログラムが終了します。

**スクリプト例**

~c
adam, _init;
init, $"Start!; _step1;
step1, $"Next Step; =data; $~step1; _end;
~e

出力例：

~a
Start!Next Stepdata
~e

repo: [[https://github.com/Soyo-Wind/Flags/]]

# Erlang Cheat Scheet
Erlang言語のチートシート。
標準モジュールについては別に扱う。

## コメント

```erlang
%% 慣習的に%を２つ使う
```



## 値

```erlang
1                        %% int
1.1                      %% float
[1, 2]                   %% list
"string"                 %% list
<<1234>>                 %% binary
<<"binary string">>      %% binary
atom                     %% rubyのsymbolと同じ、実行時に定数となる
'quoted atom'            %% atom
2#101010                 %% 42、2進数
8#0755                   %% 493、8進数
16#ff                    %% 255、16進数
true                     %% bool(atom)
{atom, 1, 1.1, "string"} %% tuple
```



## 束縛

```erlang
One.             %% variable 'One' is unbound
One = 1.         %% 初期化
One = 2.         %% exception error: no match of right hand side value 2
One = Uno = 1.   %% Oneと1の値が等しいので上記の例外が起きない
Two = One + One. %% 2
```



## 式

```erlang
3         = 1 + 2          .
1         = 2 - 1          .
6         = 2 * 3          .
2.5       = 5 / 2          .
2         = 5 div 2        .
1         = 5 rem 2        .
false     = true and false .
true      = true or false  .
true      = true xor false .
true      = not false      .
true      = 1 =:= 1        .
false     = 1 =/= 1        .
false     = 1 =:= 1.0      .
true      = 1 =/= 1.0      .
true      = 1 == 1.0       .
false     = 1 /= 1.0       .
false     = 1 < 1          .
true      = 1 >= 1         .
true      = 1 =< 1         .
[1, 2]    = [1] ++ [2]     .
[2]       = [1, 2] -- [1]  .
[1, 2, 3] = [1 | [2 | [3]]].
```



## 式の落とし穴

```erlang
true = "abc" == [97, 98, 99]. %% 文字列は配列と一緒
true = 1 < false            . %% 型の比較となる number < atom < reference < fun < port < pid < tuple < list < bit string
```



## Tuple

```erlang
Point = {1, 2}.
{X, Y} = Point.
{X, _} = Point.
```



## 配列

```erlang
1 = hd([1, 2, 3]).
[2, 3] = tl([1, 2, 3]).
[Head | Tail] = [1, 2, 3].
Head = 1.
Tail = [2, 3].
```



## リスト内包表記

```erlang
[2, 4, 6]  = [2 * N || N <- [1, 2, 3]]  . 
[1, 2]     = [N || N <- [1,2,3], N =< 2]. 
```



## バイナリ

```erlang
Color = 16#010203.
RGB = <<Color:24>>.
RGB = <<1, 2, 3>>.
<<R, G, B>> = RGB.
R = 1.
G = 2.
B = 3.
<<R:8, _/binary>> = <<Color:24>>.
<<_:8, G:8, _:8>> = <<Color:24>>.

<<X1/unsigned>> = <<-1>>.
X1 = 255.
<<X2/signed>> = <<-1>>.
X2 = -1.
<<X2/integer-signed-little>> = <<-1>>.
<<256:8/unit:4>> == <<0, 0, 1, 0>>.
```



### バイナリパターンマッチ構文
Var
Var:size
Var/type
Var:size/type

上記の意味
size      = 数値
type      = integer | float | binary | bytes | bitstring | bits | utf8 | utf16 | utf32
          | integer-(sign)-(endian)
          | unit:(unit_size)
sign      = signed | unsigned
endian    = big | little native
unit_size = 1..256



### バイナリのビット操作

```erlang
2#0010 = 2#0001 bsl 1.
2#0010 = 2#0100 bsr 1.
2#1001 = 2#0001 bor 2#1000.
```



### バイナリ内包表記

```erlang
<<2,3,4,5>> = << <<(X + 1)>> || <<X>> <= <<1,2,3,4>>>>.
<<2,4>> = << <<X>> || <<X>> <= <<1,2,3,4>>, X rem 2 == 0>>.
```



## 関数

```erlang
Add = fun(A, B) -> A + B end.
3 = Add(1, 2).

Sum = fun(X) -> (
            fun F(N, ACC) when N =< 0 -> ACC;
                F(N, ACC) -> F(N - 1, N + ACC)
            end
        )(X, 0)
    end.
55 = Sum(10).
```



## if

```erlang
Show = fun(X) -> if
            X < 0 -> "negative";
            X > 0 -> "positive";
            true -> "zero"
        end
    end.
"negative" = Show(-1).
"positive" = Show(1).
"zero" = Show(0).
```



## case

```erlang
Show = fun(N) -> case N of
            {add, A, B} -> A + B;
            {sub, A, B} -> A - B;
            {sign, 0} -> "zero";
            {sign, A} when A > 0 -> "positive";
            {sign, A} when A < 0 -> "negative";
            _ -> N
        end
    end.
3 = Show({add, 1, 2}).
1 = Show({sub, 1, 2}).
"zero" = Show({sign, 0}).
"positive" = Show({sign, 1}).
"negative" = Show({sign, -1}).
invalid = Show(invalid).
```



## モジュール

```erlang:package_future.erl
-module(package_future).
-export([inc/1]).

inc(X) -> add(X, 1).
add(A, B) -> A + B.
```

```text
# erl
> c(package_future),
> package_future:inc(1).
> package_future:add(1, 2). % exception error: undefined function package_future:add/2
```



## マクロ

```erlang:a.erl
-module(a).
-export([show/0]).

-define(PI, 3.14).
-define(ADD(X, Y), X + Y).
-ifdef(DEBUG_MODE). % erl -D DEBUG_MODE
-define(DEBUG(X), io:format("debug: ~p~n", [X])).
-else.
-define(DEBUG(_), ok).
-endif.

show() ->
    io:format("~p~n", [?PI]),
    io:format("~p~n", [?ADD(1, 2)]),
    io:format("~p~n", [?MODULE]),        % "a"
    io:format("~p~n", [?MODULE_STRING]), % "a.erl"
    io:format("~p~n", [?FILE]),
    io:format("~p~n", [?LINE]),
    io:format("~p~n", [?MACHINE]),       % BEAM
    ?DEBUG("debug information").
```

```shell
> erlc -DDEBUG_MODE a.erl && erl -noshell -s a show -s init stop
3.14
3
a
"a"
"a.erl"
18
'BEAM'
debug: "debug information"
```



## 型変換

```erlang
-spec atom_to_binary          (Atom, Encoding) -> binary()
-spec atom_to_list            (Atom) -> string()
-spec binary_to_atom          (Binary, Encoding) -> atom()
-spec binary_to_existing_atom (Binary, Encoding) -> atom()
-spec binary_to_float         (Binary) -> float()
-spec binary_to_integer       (Binary) -> integer()
-spec binary_to_integer       (Binary,Base) -> integer()
-spec binary_to_list          (Binary) -> [byte()]
-spec binary_to_list          (Binary, Start, Stop) -> [byte()]
-spec binary_to_term          (Binary) -> term()
-spec binary_to_term          (Binary, Opts) -> term()
-spec bitstring_to_list       (Bitstring) -> [byte() | bitstring()]
-spec float_to_binary         (Float) -> binary()
-spec float_to_binary         (Float, Options) -> binary()
-spec float_to_list           (Float) -> string()
-spec float_to_list           (Float, Options) -> string()
-spec erlang:fun_to_list      (Fun) -> string()
-spec integer_to_binary       (Integer) -> binary()
-spec integer_to_list         (Integer) -> string()
-spec iolist_to_binary        (IoListOrBinary) -> binary()
-spec list_to_atom            (String) -> atom()
-spec list_to_binary          (IoList) -> binary()
-spec list_to_bitstring       (BitstringList) -> bitstring()
-spec list_to_existing_atom   (String) -> atom()
-spec list_to_float           (String) -> float()
-spec list_to_integer         (String) -> integer()
-spec list_to_integer         (String, Base) -> integer()
-spec list_to_pid             (String) -> pid()
-spec list_to_tuple           (List) -> tuple()
-spec pid_to_list             (Pid) -> string()
-spec erlang:port_to_list     (Port) -> string()
-spec erlang:ref_to_list      (Ref) -> string()
-spec term_to_binary          (Term) -> ext_binary()
-spec term_to_binary          (Term, Options) -> ext_binary()
-spec tuple_to_list           (Tuple) -> [term()]
-spec integer_to_list         (Integer, Base) -> string()
-spec integer_to_binary       (Integer, Base) -> binary()
```

## 型チェック

```erlang
-spec erlang:is_builtin (Module, Function, Arity) -> boolean()
-spec is_process_alive  (Pid) -> boolean()
-spec is_atom           (Term) -> boolean()
-spec is_binary         (Term) -> boolean()
-spec is_bitstring      (Term) -> boolean()
-spec is_boolean        (Term) -> boolean()
-spec is_float          (Term) -> boolean()
-spec is_function       (Term) -> boolean()
-spec is_function       (Term, Arity) -> boolean()
-spec is_integer        (Term) -> boolean()
-spec is_list           (Term) -> boolean()
-spec is_number         (Term) -> boolean()
-spec is_pid            (Term) -> boolean()
-spec is_map            (Term) -> boolean()
-spec is_port           (Term) -> boolean()
-spec is_record         (Term,RecordTag) -> boolean()
-spec is_record         (Term,RecordTag,Size) -> boolean()
-spec is_reference      (Term) -> boolean()
-spec is_tuple          (Term) -> boolean()
```



## 例外

```erlang
1/0. % ** exception error: an error occurred when evaluating an arithmetic expression
     %    in operator  '/'/2
     %       called as 1 / 0

try
    io:format("try in~n"),
    1/0
of
    _ -> ok
catch
    error:badarith -> "zero division";
    _:_ -> "other"
after
    io:format("after~n")
end.

catch 1/0.
catch throw(reason).
catch exit(die).
3 = (catch 1 + 2).
1 = (catch (fun() ->
        throw(1),
        1 / 0
    end)()).
```



## プロセスとメッセージパッシング
```erlang:mod.erl
-module(mod).
-export([show/0, run/1, producer/2, consumer/2]).

show() ->
    Pids = registered(),
    Pid = spawn(?MODULE, run, [self()]),
    register(run, Pid),
    io:format("waiting for message processes:~p~n", [registered() -- Pids]),
    receive
        Message -> io:format("result ~p~n", [Message])
    after 1000 ->
		io:format("timeout~n", [])
    end,
    io:format("done processes:~p~n", [registered() -- Pids]).

run(FromPid) ->
    Count = 3,
    Pid1 = spawn_link(?MODULE, consumer, [self(), 0]),
    Pid2 = spawn(?MODULE, producer, [Count, Pid1]),
    register(consumer, Pid1),
    register(producer, Pid2),
    receive
        Message -> FromPid ! Message
    end.

producer(0, Pid) ->
    io:format("producer is done"),
    Pid ! eof;
producer(Count, Pid) ->
    io:format("produce ~p~n", [Count]),
    Pid ! {add, Count},
    producer(Count - 1, Pid).

consumer(Pid, Acc) ->
    receive
        {add, Count} ->
            io:format("consume ~p~n", [Count]),
            consumer(Pid, Acc + Count);
        eof ->
            Pid ! Acc;
		Unexpected ->
			io:format("skip consume ~p~n", [Unexpected])
    end.
```erlang

```shell
# erl
1> c(mod), mod:show().
waiting for message processes:[run]
produce 3
produce 2
consume 3
produce 1
consume 2
producer is doneconsume 1
result 6
done processes:[]
ok
```

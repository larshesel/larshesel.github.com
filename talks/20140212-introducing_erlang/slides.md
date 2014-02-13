% title: Introducing Erlang
% subtitle: - What the fuzz is all about
% author: Lars Hesel Christensen
% author: @larshesel on Twitter
% thankyou: Thank you!
% thankyou_details: Thanks to <a href="http://redbooth.com">Redbooth.com</a> for hosting!
% contact: <a href="http://larshesel.dk">larshesel.dk</a>
% contact: <a href="http://github.com/larshesel">github.com/larshesel</a>
% contact: <a href="http://twitter.com/larshesel">@larshesel on Twitter</a>
% favicon: /favicon.ico

---
title: This talk

- Yours truly
- Why Erlang?
- Erlang philosophy
- A quick tour
- Fault tolerance
- OTP
- The community
- Maybe have a look at maps in R17-rc1
- Outro

---
title: A bit about me

- From Denmark
- Lived in Switzerland 2008-2013
- In Barcelona since October

<br />

- Embedded devices (c, c++) for telecom and industry
- Desktop applications (.Net) for reinsurance
- Android apps (Java) for banks and industry
- Server side (Erlang) for retail

---
title: Why Erlang?

Ericsson had (among others) the following requirements:

- Thousands of concurrent tasks
- Fault tolerance
- Continuous operation
- Soft real time
- Distribution
- Very large systems (millions of SLOC)

<!-- Mid '80s
    Ericsson trying to make better products
    Joe Armstrong, Mike Williams, Robert Virding
    task: "do something about how we write software"
    -->
---
title: Erlang pocket philosophy

- The world is parallel
- Isolation
- Recoverability
- Let it crash (defer error handling)
- ...

Were the first that came to mind...

<!-- that the world is concurrent - this had a huge impact on the language design
     when implementing real world domains (as telephony systems)
     we need concurrency.
     It's how our brain works.
     We do not share memory - we send messages. -->
<!-- Implications: -->
<!-- - Don't code defensively (except at data entry points) -->

---
title: Erlang the language

- Impure functional language
- Single assignment, immutable
- Strict evaluation
- Dynamic (with type annotations & Dialyzer)
- <b>Actors</b>
- <b>Message passing</b>

<br />

- No classes or inheritance!

---
title: Functions

<pre class="prettyprint" data-lang="erlang">
-module(fibonacci).
-export([fibonacci/1]).

fibonacci(1) -> 1;
fibonacci(2) -> 1;
fibonacci(N) -> fibonacci(N-1) + fibonacci(N-2).
</pre>


<pre class="prettyprint" data-lang="erlang">
> fibonacci:fibonacci(1).
1
> fibonacci:fibonacci(3).
2
> fibonacci:fibonacci(10).
55
</pre>


---
title: Case

<pre class="prettyprint" data-lang="erlang">
-module(using_case).
-export([fibonacci/1]).

fibonacci(N) ->
    case N of
        1 -> 1;
        2 -> 1;
        _ -> fibonacci(N-1) + fibonacci(N-2)
    end.
</pre>

<pre class="prettyprint" data-lang="erlang">
> using_case:fibonacci(1).
1
> using_case:fibonacci(3).
2
> using_case:fibonacci(10).
55
</pre>

---
title: Pattern matching

<pre class="prettyprint" data-lang="erlang">
> {_, Second, Third, _, _} = {a, tuple, with, five, entries}.
{a,tuple,with,five,entries}
> Second.
tuple
> Third.
with
</pre>

- evaluating a function call
- case statements
- receive statements
- try statements
- and using <code>=</code> as above

---
title: Pattern matching

Consider the function:

<pre class="prettyprint" data-lang="erlang">
pattern_match({_, Second, Third, _, _}) ->
    [Second, Third].
</pre>

<pre class="prettyprint" data-lang="erlang">
> pattern_match({a, tuple, with, five, entries}).
[tuple,with]
</pre>

---
title: Canonical pattern matching example

<pre class="prettyprint" data-lang="erlang">
-define(IP_VERSION, 4).
-define(IP_MIN_HDR_LEN, 5).

DgramSize = byte_size(Dgram),
case Dgram of
    &lt;&lt;?IP_VERSION:4, HLen:4, SrvcType:8, TotLen:16,
      ID:16, Flgs:3, FragOff:13,
      TTL:8, Proto:8, HdrChkSum:16,
      SrcIP:32,
      DestIP:32, RestDgram/binary&gt;&gt; when HLen &gt;= 5, 4*HLen =&lt; DgramSize ->
        OptsLen = 4*(HLen - ?IP_MIN_HDR_LEN),
        &lt;&lt;Opts:OptsLen/binary,Data/binary&gt;&gt; = RestDgram,
    ...
end.
</pre>

---
title: Messages

<pre class="prettyprint" data-lang="erlang">
spawn_a_fun() ->
    Pid = spawn(fun rec_fun/0),
    Pid ! {packet, "Hello!"},
    ok.

rec_fun() ->
    receive
        {packet, Msg} ->
            io:format("Got msg: ~p~n", [Msg])
    end.
</pre>

<pre class="prettyprint" data-lang="erlang">
> spawn_a_fun().
Got msg: "Hello!"
ok
</pre>

---
title: Fault tolerance, supervisors
class: img-top-center

<img height=400 src=figures/supervisor.png style />

---
title: Supervising supervisors
class: img-top-center

<img height=400 src=figures/supervision_tree.png style />

---
title: Fault tolerance
class: img-top-center

<img height=400 src=figures/shared_mem_ok.png style />

---
title: Fault tolerance
class: img-top-center

<img height=400 src=figures/shared_memory_crash.png style />

---
title: Fault tolerance
class: img-top-center

<img height=400 src=figures/shared_memory_whatnow.png style />

The remaining process cannot meaningfully continue!

<!-- since the crashed process couldn't anticipate the crash,
    how can you expect the other process to be able to understand
    the corrupt or undefined state and continue? -->

---
title: Actors & message passing!

Actors are

<img height=300 src=figures/message_passing.png style="position: absolute; top: 15%; right: 10%;" />

- Process
- State and msgbox

Actors encapsulate

- Actors can only change their own state
- Change state of others indirectly, through message passing

<!-- you can't have fault tolerance of concurrent activities
    in a shared memory model! -->
<!-- OR, you can't convince anyone that you have it. You can with actors and message passing! -->

---
title: The Erlang VM

- Implements processes independently of OS
- Supports massive concurrency
- Fast process creation. I once measured $~7\mu s$/process when creating 1M processes
- Fast context switching
- Rock solid

---
title: Let it crash philosophy

- Exception: The run-time system does not know what to do
- Error: The programmer does not know what to do

Let's look at an error example (from <a
href="http://www.erlang.org/download/armstrong_thesis_2003.pdf">Joe's
Ph.D thesis</a>).

<br />
Assume spec says we handle two op-codes:

<pre class="prettyprint" data-lang="erlang">
asm(load) -> 1;
asm(store) -> 2.
</pre>

---
title: Let it crash (cont.)

What should happen if we eval <code>asm(jump)</code>?

<pre class="prettyprint" data-lang="erlang">
asm(load) -> 1;
asm(store) -> 2;
asm(X) -> ????????????
</pre>

We could try:

<pre class="prettyprint" data-lang="erlang">
asm(load) -> 1;
asm(store) -> 2;
asm(X) -> exit({oops,i,did,it,again,in,asm,X})
</pre>

But this isn't really an improvement!

---
title: Let it crash (cont.)

So don't! Just let it crash and let the supervisor take care of it.

<pre class="prettyprint" data-lang="erlang">
asm(load) -> 1;
asm(store) -> 2;
</pre>

This is much easier to understand. No clutter!

---
title: OTP middleware

- a standard library
- servers (gen_server)
- finite state machines (gen_fsm)
- event handling (gen_event)
- testing, tracing, monitoring
- and much, much more...

---
title: R17 maps

... Let's all go to the terminal

---
title: The community

- mailing lists (questions, patches, bugs)
- IRC
- conferences (EUC, EFs, Code Mesh, ...)
- meetups like this
- ...

---
title: References:

- <a href="http://www.erlang.org">Erlang Programming Language - erlang.org</a>
- <a href="http://www.erlang.org/download/armstrong_thesis_2003.pdf">Joe Armstrong's Ph.d Thesis</a>
- <a href="http://erlang.org/pipermail/erlang-questions/">Erlang Questions Mailing List</a>
- <a href="http://learnyousomeerlang.com/">Learn You Some Erlang</a>
- <a href="http://dl.acm.org/citation.cfm?doid=1810891.1810910">Erlang, Joe Armstrong</a>

---
title: Erlang humor / outro

- The hipster version: <a href="http://www.youtube.com/watch?v=rRbY3TMUcgQ">Erlang the Movie II</a>

- The originial: <a href="http://www.youtube.com/watch?v=xrIjfIjssLE">Erlang the Movie</a>

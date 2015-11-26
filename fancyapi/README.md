# erlang api sample

## requirement

rebar (https://github.com/rebar/rebar/wiki)  
GettingStarted (https://github.com/rebar/rebar/wiki/Getting-started)

## reference

following site  
http://smyck.net/2013/02/17/how-to-set-up-a-basic-http-api-with-erlang-in-10-minutes/

## sample making

### step1 preparing

    $ mkdir fancyapi
    $ cd fancyapi
    $ rebar create-app appid=fancyapi

### step2 make rebar.config and solve dependency

see also

* https://github.com/wooga/etest  
* https://github.com/wooga/etest_http  
* https://github.com/knutin/elli

make rebar.config quote from etest README

    % Compiler Options for rebar
    {erl_opts, [
        {src_dirs, ["src", "test"]}
    ]}.

    % Dependencies
    {deps, [
        {etest, ".*", {git, "git://github.com/wooga/etest.git"}}
    ]}.

    % Which files to cleanup when rebar clean is executed.
    {clean_files, ["ebin/*.beam"]}.

add dependency for etest_http and elli

    % Compiler Options for rebar
    {erl_opts, [
        {src_dirs, ["src", "test"]}
    ]}.

    % Dependencies
    {deps, [
        {etest, ".*", {git, "git://github.com/wooga/etest.git"}},
        {etest_http, "", {git, "git://github.com/wooga/etest_http.git"}},       <- added
        {elli, "", {git, "git://github.com/knutin/elli.git"}}                   <- added
    ]}.

    % Which files to cleanup when rebar clean is executed.
    {clean_files, ["ebin/*.beam"]}.

fetch dependency

    $ rebar get-deps

when finished.

    $ tree -L 2
    .
    |-- README.md
    |-- deps
    |   |-- elli
    |   |-- etest
    |   |-- etest_http
    |   `-- jiffy
    |-- rebar.config
    `-- src
        |-- fancyapi.app.src
        |-- fancyapi_app.erl
        `-- fancyapi_sup.erl

    6 directories, 5 files

### step3 make test case

make test directory

    $ mkdir test

quote test code from etest_http and make file and edit and add some codes

    $ vi test/my_fancyapi_test.erl
    -module (my_fancyapi_test).
    -compile (export_all).

    % etest macros
    -include_lib ("etest/include/etest.hrl").
    % etest_http macros
    -include_lib ("etest_http/include/etest_http.hrl").

    before_suite() ->
        application:start(fancyapi).

    after_suite() ->
        application:stop(fancyapi).

    test_basic_operation() ->
        ?assert_equal(1, 1).

    test_hello_world() ->
        Response = ?perform_get("http://localhost:3000/hello/world"),
        ?assert_status(200, Response),
        ?assert_body_contains("Hello", Response),
        ?assert_body("Hello World!", Response).

### step4 edit supervisor

quote sample code from elli and edit fancyapi_sup.erl

    -module(fancyapi_sup).

    -behaviour(supervisor).

    %% API
    -export([start_link/0]).

    %% Supervisor callbacks
    -export([init/1]).

    %% Helper macro for declaring children of supervisor
    -define(CHILD(I, Type), {I, {I, start_link, []}, permanent, 5000, Type, [I]}).

    %% ===================================================================
    %% API functions
    %% ===================================================================

    start_link() ->
        supervisor:start_link({local, ?MODULE}, ?MODULE, []).

    %% ===================================================================
    %% Supervisor callbacks
    %% ===================================================================

    init([]) ->
        ElliOpts = [{callback, fancyapi_callback}, {port, 3000}],   <- quote from elli sample
        ElliSpec = {
            fancy_http,
            {elli, start_link, [ElliOpts]},
            permanent,
            5000,
            worker,
            [elli]},

        {ok, { {one_for_one, 5, 10}, [ElliSpec]} }.

### step5 make callback module

quote from elli sample again and edit

    $ vi src/fancyapi_callback.erl
    -module(fancyapi_callback).
    -export([handle/2, handle_event/3]).

    -include_lib("elli/include/elli.hrl").
    -behaviour(elli_handler).

    handle(Req, _Args) ->
        %% Delegate to our handler function
        handle(Req#req.method, elli_request:path(Req), Req).

    handle('GET',[<<"hello">>, <<"world">>], _Req) ->
        %% Reply with a normal response. 'ok' can be used instead of '200'
        %% to signal success.
        {ok, [], <<"Hello World!">>};

    handle(_, _, _Req) ->
        {404, [], <<"Not Found">>}.

    %% @doc: Handle request events, like request completed, exception
    %% thrown, client timeout, etc. Must return 'ok'.
    handle_event(_Event, _Data, _Args) ->
        ok.

### step6 compile and run test

    $ rebar compile && deps/etest/bin/etest-runner
    ...
    ...
    =========================================
      Failed: 0.  Success: 2.  Total: 2.

### step7 run on prompt and access from http

boot application

    $ erl -pa deps/*/ebin ebin
    1> application:start(fancyapi).
    ok
    2> application:which_applications().
    [{fancyapi,[],"1"},
     {stdlib,"ERTS  CXC 138 10","2.6"},
     {kernel,"ERTS  CXC 138 10","4.1"}]

now you can access from http://localhost:3000/hello/world

    $ curl http://localhost:3000/hello/world
    Hello World!

### step8 make shortcut start() have no arguments

edit fancyapi_app.erl

    -module(fancyapi_app).

    -behaviour(application).

    %% Application callbacks
    -export([start/0, start/2, stop/1]).

    %% ===================================================================
    %% Application callbacks
    %% ===================================================================

    start() ->
        application:start(fancyapi).

    start(_StartType, _StartArgs) ->
        fancyapi_sup:start_link().

    stop(_State) ->
        ok.

### step9 boot from command line

    $ erl -detached -pa deps/*/ebin ebin -s fancyapi_app start
    $ ps aux | grep beam
    root      4293  1.3  1.4 369288 14968 ?        Sl   16:38   0:00 /usr/local/lib/erlang/erts-7.1/bin/beam .....


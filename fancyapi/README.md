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

### step2 make rebar.config

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


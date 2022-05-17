# power_shell

Advanced system-wide Erlang shell capabilities. Evaluates Erlang code loaded from the module debug
information, or source file.

Build with rebar3:

    rebar3 compile

Versions supported: tested with 21, 22, 23 and 24.

Documentation can be built with edoc: `rebar3 edoc`

## License
power_shell license is available in the LICENSE file in the root directory of this source tree.

## Usage

power_shell can be used as an application, and in form of a library. When started as application, power_shell examines
**shell_integration** configuration parameter. It can be set to shell_default, or user_default. Depending on the
parameter, power_shell can load itself as user_default module (if none was loaded before), or add proxy calls to
already loaded user_default module.

### Evaluating non-exported functions
Enter `power_shell:eval(Mod, Fun, [Arg1, Arg2, Arg3]).` in the shell to evaluate function Fun in module Mod. It does 
not need to be exported. 
If module Mod is currently loaded, power_shell will try to locate corresponding *.beam file, load debug info chunk and 
evaluate function Fun with supplied arguments. 
If module is not loaded, but *.beam file can be found, power_shell does not attempt to load it. However, if there is a 
remote (fully qualified) call, Erlang VM (BEAM) will load the file automatically.

When shell integration (injection) is enabled, evaluating non-exported functions is simplified to just `eval(Mod, Fun, [Args].`

### Debugging shortcuts
Available with shell integration.

### Coverage Analysis Tool for Erlang
Modules that were patched by 'cover' application are also supported. If a module was cover_compiled, and it is being
evaluated with power_shell:eval, coverage analysis works as expected.

### -on_load() attribute support
To preserve compatibility with earlier versions, power_shell does not execute -on_load() by default. To change
this behaviour, set `skip_on_load` application variable to `false`:

    ok = application:load(power_shell),
    ok = application:set_env(power_shell, skip_on_load, false).

### Recompiling and hot-loading modules with extra functions exported
Starting with version 1.2.0, power_shell can recompile an existing module exporting all or selected functions. Use
`power_shell:export(Mod)` to export all functions. This may be used in conjunction with Common Test suites,
allowing for white-box testing of functions that should not be exported in production code. Example:

    my_case(Config) when is_list(Config) ->
        Old = power_shell:export(prod_module),
        WhiteBoxRet = prod_module:not_exported_fun(123),
        power_shell:revert(Old).

For reliable Common Test execution `export/1,2,3` start a linked process that will reload the original code
upon termination. This allows to avoid failing other test cases that do not expect extra exports available.

## Configuration
During application startup, power_shell examines following application environment variables:
 
- **cache_code** (boolean, default false) - start code cache process. This process keeps decompiled beam files in 
memory, speeding up execution if code does not change
- **shell_integration** (atom, shell_default | user_default, default is user_default) - apply shell integration, 
making `power_shell:eval(Mod, Fun, Args)` available in shell, `eval(Mod, Fun, Args)`
- **skip_on_load** (boolean, default true) - skip -on_load() execution when decompiling beam (or loading source)

It is possible to supply values using sys.config file, or via command line when starting BEAM:

    erl -power_shell cache_code true

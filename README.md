# Command-Option-Argument

COA is a yet another parser for command line options.
You can choose one of the [existing modules](https://github.com/joyent/node/wiki/modules#parsers-commandline),
or write your own like me.

## Examples

````javascript
require('coa').Cmd() // main (top level) command declaration
    .name(process.argv[1]) // set top level command name from program name
    .title('My awesome command line util') // title for use in text messages
    .helpful() // make command "helpful", i.e. options -h --help with usage message
    .opt() // add some option
        .name('version') // name for use in API
        .title('Version') // title for use in text messages
        .short('v') // short key: -v
        .long('version') // long key: --version
        .type(Boolean) // type Boolean for options without value
        .act(function(opts) { // add action for option
            this.exit( // exit program with code 0 and text message
                JSON.parse(require('fs').readFileSync(__dirname + '/package.json'))
                    .version);
        })
        .end() // end option chain and return to main command
    .cmd().name('subcommand').apply(require('./subcommand').COA).end() // load subcommand from module
    .cmd() // inplace subcommand declaration
        .name('othercommand').title('Awesome other subcommand').helpful()
        .opt()
            .name('input').title('input file, required')
            .short('i').long('input')
            .validate(function(v) { // validator function, also for translate simple values
                return require('fs').createReadStream(v) })
            .required() // make option required
            .end() // end option chain and return to command
        .end() // end subcommand chain and return to parent command
    .parse(process.argv.slice(2)); // parse and run on process.argv
````

````javascript
// subcommand.js
exports.COA = function() {
    this
        .title('Awesome subcommand').helpful()
        .opt()
            .name('output').title('output file')
            .short('o').long('output')
            .output() // use default preset for "output" option declaration
            .end()
};
````

## API

### Cmd
Command is a top level entity. Commands may have options and arguments.

#### Cmd.name
Set a canonical command identifier to be used anywhere in the API.<br>
**@param** *String* `_name` command name<br>
**@returns** *COA.Cmd* `this` instance (for chainability)

#### Cmd.title
Set a long description for command to be used anywhere in text messages.<br>
**@param** *String* `_title` command title<br>
**@returns** *COA.Cmd* `this` instance (for chainability)

#### Cmd.cmd
Create new subcommand for current command.<br>
**@returns** *COA.Cmd* `new` subcommand instance

#### Cmd.opt
Create option for current command.<br>
**@returns** *COA.Opt* `new` option instance

#### Cmd.arg
Create argument for current command.<br>
**@returns** *COA.Opt* `new` argument instance

#### Cmd.act
Add (or set) action for current command.<br>
**@param** *Function* `act` action function,
    invoked in the context of command instance
    and has the parameters:<br>
        - *Object* `opts` parsed options<br>
        - *Array* `args` parsed arguments<br>
**@param** *{Boolean}* [force=false] flag for set action instead add to existings<br>
**@returns** *COA.Cmd* `this` instance (for chainability)

#### Cmd.apply
Apply function with arguments in context of command instance.<br>
**@param** *Function* `fn`<br>
**@param** *Array* `args`<br>
**@returns** *COA.Cmd* `this` instance (for chainability)

#### Cmd.helpful
Make command "helpful", i.e. add -h --help flags for print usage.<br>
**@returns** *COA.Cmd* `this` instance (for chainability)

#### Cmd.errorExit
Terminate program with error code 1.
**@param** *String* `msg` message for print to STDERR<br>
**@param** *{Object}* [o] optional object for print with message

#### Cmd.exit
Terminate program with error code 0.<br>
**@param** *String* `msg` message for print to STDERR

#### Cmd.usage
Build full usage text for current command instance.<br>
**@returns** *String* `usage` text

#### Cmd.parse
Parse arguments from simple format like NodeJS process.argv.<br>
**@param** *Array* `argv`<br>
**@returns** *COA.Cmd* `this` instance (for chainability)

#### Cmd.end
Finish chain for current subcommand and return parent command instance.<br>
**@returns** *COA.Cmd* `parent` command

### Opt
Option is a named entity. Options may have short and long keys for use from command line.<br>
**@namespace**<br>
**@class** Presents option

#### Opt.name
Set a canonical option identifier to be used anywhere in the API.<br>
**@param** *String* `_name` option name<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.title
Set a long description for option to be used anywhere in text messages.<br>
**@param** *String* `_title` option title<br>
**@returns** *COA.Opt* `this` instance (for chainability)


#### Opt.short
Set a short key for option to be used with one hyphen from command line.<br>
**@param** *String* `_short`<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.long
Set a short key for option to be used with double hyphens from command line.<br>
**@param** *String* `_long`<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.type
Set a type of option. Mainly using with Boolean for options without value.<br>
**@param** *Object* `_type`<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.arr
Makes an option accepts multiple values.<br>
Otherwise, the value will be used by the latter passed.<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.required
Makes an option required.<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.validate
Set a validation function for option.<br>
Value from command line passes through before becoming available from API.<br>
**@param** *Function* `_validate` validating function,
    invoked in the context of option instance
    and has one parameter with value from command line<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.def
Set a default value for option.
Default value passed through validation function as ordinary value.<br>
**@param** *Object* `_def`<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.output
Make option value outputing stream.<br>
It's add useful validation and shortcut for STDOUT.<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.act
Add action for current option command.
This action is performed if the current option
is present in parsed options (with any value).<br>
**@param** *Function* `act` action function,
    invoked in the context of command instance
    and has the parameters:<br>
        - *Object* `opts` parsed options<br>
        - *Array* `args` parsed arguments<br>
**@returns** *COA.Opt* `this` instance (for chainability)

#### Opt.end
Finish chain for current option and return parent command instance.<br>
**@returns** *COA.Cmd* `parent` command


### Arg
Argument is a unnamed entity.<br>
From command line arguments passed as list of unnamed values.

#### Arg.name
Set a canonical argument identifier to be used anywhere in text messages.<br>
**@param** *String* `_name` argument name<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.title
Set a long description for argument to be used anywhere in text messages.<br>
**@param** *String* `_title` argument title<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.arr
Makes an argument accepts multiple values.<br>
Otherwise, the value will be used by the latter passed.<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.required
Makes an argument required.<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.validate
Set a validation function for argument.<br>
Value from command line passes through before becoming available from API.<br>
**@param** *Function* `_validate` validating function,
    invoked in the context of argument instance
    and has one parameter with value from command line<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.def
Set a default value for argument.
Default value passed through validation function as ordinary value.<br>
**@param** *Object* `_def`<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.output
Make argument value outputing stream.<br>
It's add useful validation and shortcut for STDOUT.<br>
**@returns** *COA.Arg* `this` instance (for chainability)

#### Arg.end
Finish chain for current option and return parent command instance.<br>
**@returns** *COA.Cmd* `parent` command


## TODO
* Program API for use COA-covered programs as modules
* Shell completion
* Localization
* Shell-mode
* Configs
  * Aliases
  * Defaults

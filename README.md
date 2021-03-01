# hazel
Preprocessor for JavaScript, written in PHP.
Package includes the following files...

* hazel.php ............. plain hazel program
* hazelr.php ............ with reporting feature
* hazel-samples.php ..... sample function hooks
* hz.php ................ sample invocation
* def.hz ................ sample shortcuts

The sample invocation is a basic PHP script that includes
hazel, then fetches files with a glob from the querystring.
If the destination is a folder, all .hz files within will be 
included in the output. Usage...

```html
<script type="text/javascript" src="hz.php?my/file.hz"></script>
<script type="text/javascript" src="hz.php?my/scripts/"></script>
<script type="text/javascript" src="hz.php?api/class-*.hz"></script>
```

The sample invocation also includes samples.php and runs
hazel() over def.hz prior to loading the request. This is
how you would define your own set of custom function hooks
and common macros to all your projects. I included my own
with this package as examples, but I would assume anyone 
using Hazel would want to make their own shortcuts.

Make copies of and edit hz.php to suit your needs.

Hazel (c) 2017-2021 - 1002px.com - released under MIT license.

## Using Hazel

Hazel uses different types of macro shortcuts to truncate
javascript (or any similar language) to your specification.
Shortcuts all take the form (slash)(slash)(something)(tab),
and allow you to alter the syntax of javascript with match
macros, or wrappers that transliterate function calls and 
new structural blocks resembling for(){} or while(){}...

```javascript
//=	exact match macro
//s	macros exclusively in strings
//v	macros without : after or . before
//f	function wrapper template
//b	block wrapper template
//p	procedure hook for function wrapper
//k	procedure hook for block wrapper
//i	include another .hz file
//j	include unaltered script
//c	define starter for comment line
```

You may use `##` instead of `//` if you want to preserve 
syntax highlighting on the terms. If you are not minifying
the code, these will be turned into `//`.

## Examples

```javascript
//b	after	setTimeout(function(){#B},#0);

// USE...

after(1000){ do.something(); }
```

```javascript
##b	forln	for(#1=0;#1<#0.length;#1++){#B}

// USE...

forln(arr;i){ do.something(arr[i]); }
```

```javascript
//=	(<>	document.createElement(
//v	BODY	document.body
//=	..AP	.appendChild
//=	..IH	.innerHTML
//=	..SA	.setAttribute
//=	..SC(	.setAttribute('class',

// USE...

var EL = (<>'div');
BODY..AP(EL);
EL..IH = "Hello World.";
EL..SA('id','ex-002');
EL..SC("example");
```

Any block wrapper you create may also have an _else_ 
clause associated with it. Use `#E` in your template
to include the else clause somewhere.

```javascript
##b	forln	if(!#0||!#0.length||#0.length==0){#E}
##_             else { for(#1=0;#1<#0.length;#1++){#B} }

// USE...

forln(arr;i){ do.something(arr[i]); }
else { do.something_else(); }
```

## Template Arguments

Templates for function and block wrappers allow
numbered arguments `#0`, `#1`... `#99`. You may 
also use any of the following values as of this 
version...

```
#C	Argument count
#A	All arguments separated by ,
#N	Shift argument from left
#X	Pop argument from right
#R	Remainder of arguments
#B	Block for block wrappers
#E	Else block for block wrappers
#I	Instance number (0-based)
```

The numbered arguments are resolved first, then
the `#N` and `#X` commands in order, then the 
rest of the commands are plugged in afterwards.
`#A` will supply _all arguments regardless, 
separated by commas_, `#R` will only supply the
arguments not caught by other commands.

## Function wrapper procedure hooks

It is very easy to write PHP functions to handle 
function and block wrappers. This allows you the
ability to add server-side functions to hazel.
The hazel-samples.php file contains a number of
sample function and block wrapper hooks to use 
for inspiration.

```javascript
//p	mfn	my_hazel_function

mfn(arg1,arg2);
```

Use the `//p` hazel command to define a function
wrapper procedure. Then define the alias and the
name of the PHP function that handles this wrapper.

```php
function my_hazel_function($ind,$arg){
	return "some code";
}
```

This function receives the arguments separated by `,`
as individual strings. Hazel tracks bracket depth and
is wise to _strings_ and _regular expressions_ and can
accurately find the correct `,`s to split by.

## Block wrapper procedure hooks

Similarly, you may make custom block wrappers.

```javascript
//k	mbk	my_hazel_block

mfn(arg1;arg2){ Code... } else { Code... }
```

The PHP function handling the block gets two more 
arguments, for the `#B` block and `#E` else block.

```php
function my_hazel_function($ind,$arg,$blk,$els){
	return "some code";
}
```

The `else` clauses are entirely optional. They are
checked for with every custom block, but if you don't
use them, the clause is just ignored.

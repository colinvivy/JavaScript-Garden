## Automatic semicolon insertion

Although JavaScript has C style syntax, it does **not** enforce the use of
semicolons in the source code, it is possible to omit them.

But JavaScript is not a semicolon-less language, it in fact needs the 
semicolons in order to understand the source code. Therefore the JavaScript
parser **automatically** inserts them whenever it encounters a parse
error due to a missing semicolon.

    var foo = function() {
    } // parse error, semicolon expected
    test()

Insertion happens, and the parser tries again.

    var foo = function() {
    }; // no error, parser continues
    test()

The automatic insertion of semicolon is considered to be one of **biggest**
design flaws in the language as it *can* change the behavior of code.

### How it works

The code below has no semicolons in it, so it is up to the parser to decide where
to insert them.

    (function(window, undefined) {
        function test(options) {
            log('testing!')

            (options.list || []).forEach(function(i) {

            })

            options.value.test(
                'long string to pass here',
                'and another long string to pass'
            )

            return
            {
                foo: function() {}
            }
        }
        window.test = test

    })(window)

    (function(window) {
        window.someLibrary = {}

    })(window)

Below is the result of the parser's "guessing" game.

    (function(window, undefined) {
        function test(options) {

            // Not inserted, lines got merged
            log('testing!')(options.list || []).forEach(function(i) {

            }); // <- inserted

            options.value.test(
                'long string to pass here',
                'and another long string to pass'
            ); // <- inserted

            return; <- inserted, breaks the return statement
            { 
                foo: function() {} 
            }; // <- inserted
        }
        window.test = test; // <- inserted

    // The lines got merged again
    })(window)(function(window) {
        window.someLibrary = {}; //<- inserted

    })(window); //<- inserted

The parser drastically changed the behavior of the code above, in certain cases
it does the **wrong** thing.

### Leading parenthesis

In case of a leading parenthesis, the parse will **not** insert a semicolon.

    log('testing!')
    (options.list || []).forEach(function(i) {})

This code gets transformed into one line.

    log('testing!')(options.list || []).forEach(function(i) {})

Chances are **very** high that `log` does **not** return a function, therefore the
above will yield `TypeError` saying that `undefined is not a function`.

### Broken `return` statements

The JavaScript parse also does not correctly handle return statements which are
followed by a new line. 

    return;
    { // gets interpreted as a block

        // a label and a single expression statement
        foo: function() {} 
    };

Instead it produces the above, that is simply a silent error

### In conclusion

It is highly recommended to **never** omit semicolons, it is also advocated to 
keep braces on the same line with their corresponding statements and to never omit 
them for one single-line `if` / `else` statements. Both of these measures will 
not only improve the consistency of the code, they will also prevent the 
JavaScript parser from changing its behavior.

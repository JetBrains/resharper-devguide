[//]: # (title: Caching Lexers)

The root node in the PSI parse tree is `IFile`, but the implementation of this interface should also implement `IFileImpl`. This exposes properties and methods that are important for the implementation of `IFile`, one of which is `TokenBuffer`.

The `TokenBuffer` property is an optional cache of the tokens in a file. If not-`null`, it contains the start and end offset, lexer state and type of all of the tokens in the file. The constructor will take in an `ILexer`, and it will immediately scan the whole file and store the processed tokens.

The `CachingLexer` class implements `ILexer` on top of a `TokenBuffer`, using the tokens from this cache. Under typical use, this offers little benefit over simply using the lexer directly - a lexer is usually efficiently implemented with a series of [lookup tables](http://www.cs.man.ac.uk/~pjj/cs212/ho/node6.html). The benefit comes with incremental parsing.

When a user edits a file, the PSI tree needs to be updated. To reduce the impact of this, an incremental parser will only parse the range of the file that has changed, and update the corresponding sub-tree in the PSI. For example, a change inside a C# method body will only re-parse that method body, and not the class definition or other methods in the file.

Of course, if part of a file has changed and needs to be re-parsed, the underlying tokens need to be re-lexed, too. The `TokenBuffer.Rescan` method will create a new lexer and re-scan the file, returning a new instance of `TokenBuffer` with an updated cache of tokens. An incremental parser will use a `CachedLexer` that uses this `TokenBuffer` to re-parse the affected region.

The `Rescan` method will try to optimise re-scanning the file and creating a new buffer of tokens. If the underlying lexer implements `IIncrementalLexer`, it will first copy the unchanged tokens at the start of the buffer, and then call `IIncrementalLexer.Start` to restart the lexer at the offset of the change, avoiding processing the unchanged part at the start of the file. This requires passing in the offset and the state of the lexer at that location, as returned by `ILexerEx.LexerStateEx` during the initial build of the token buffer cache, and stored in `TokenBuffer`. This changed section is then copied into the new `TokenBuffer`. The `TokenBuffer` will attempt to re-synchronise with the existing token buffer, and if possible, copy the tail end of the tokens into the new buffer. This means only the changed portion of the file is re-scanned, and the tokens at the start and end of the file are reused.

 >  The `IIncrementalLexer` and `ILexerEx` interfaces are somewhat biased towards lexers created with [CsLex](CsLex.md), as the state is a `uint` value, and this is how CsLex maintains its internal state. However, the value could be used to map to another (static) value or used as bit fields to map state, as the `CSharpLexerStatePacker` class does.
 >
 {type="note"}

If the underlying lexer does not implement `IIncrementalLexer`, the `Rescan` method will start the lexer from the start of the file, and re-lex the entire file, from the beginning.

If a parser supports incremental parsing, it should use an instance of `CachingLexer` as its lexer - the `ToCachingLexer` extension method on `ILexer` will create this for you.

More detail is provided in the section on incremental parsing.
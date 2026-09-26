# لغة وِب — Arabic Web Compiler (minimal)

An Arabic-first programming language and compiler that transpiles `.arweb` source files into standalone HTML/CSS/JavaScript. Built with ANTLR4, with Arabic-language error reporting and optional runtime error capture in a real browser.

This is the minimal/trimmed build of the compiler: a single grammar drives lexing, parsing, semantic analysis, and code generation end to end.

## Features

- **One grammar, three embedded languages** — `ArabicHtmlLexer` / `ArabicHtmlParser` tokenize and parse HTML-, CSS-, and JS-equivalent sections within a single `.arweb` file, switching lexer modes (`HTML`, `CSS`, `JS`, `ATTR`) as needed.
- **Full pipeline** — lexing → parsing → AST construction → semantic analysis → code generation, each stage able to short-circuit on error before the next runs.
- **Arabic error reporting** — syntax, semantic, and (optionally) runtime errors are collected, translated, and written into one merged, human-readable Arabic report (`report.txt`), grouped by phase and sorted by source location.
- **Runtime error capture** — optionally opens the generated HTML in headless Chrome via Selenium and captures real browser console errors, mapping them back to source locations and merging them into the same unified report.
- **AST inspection** — a sample parsed AST is included (`sample_ast`, `sample_ast.png`) for reference.

## Project structure

```
.
├── Grammar/          # ANTLR4 grammar and generated lexer/parser (ArabicHtmlLexer, ArabicHtmlParser)
├── AST/              # AST node visitor (ast_visitor)
├── semantic/         # Semantic analysis (SemanticAnalyzerVisitor)
├── codegen/          # Code generation to HTML/CSS/JS, error translation to Arabic
├── tests_5codes/     # Sample .arweb test programs
├── tools/            # Supporting scripts/utilities
├── generated/        # Generated ANTLR artifacts
├── output/           # Default output directory for compiled HTML/CSS/JS + reports
├── compiler_errors.py    # Error types, phases, and severities
├── main.py               # CLI entry point — runs the full compile pipeline
├── test_lexer.py         # Standalone lexer smoke test
├── test_parser.py        # Standalone parser smoke test
├── run_tests.py          # Runs the test suite against tests_5codes/
├── token_names_ar.py     # Arabic display names for lexer tokens
├── requirements.txt
├── arabic_web_compiler_documentation.docx
└── arweb_compiler.pptx
```

## Requirements

- Python 3.x
- Google Chrome (only needed for `--capture-errors`)

Install dependencies:

```bash
pip install -r requirements.txt
```

`requirements.txt` includes:

```
antlr4-python3-runtime==4.13.2
llvmlite==0.47.0
selenium>=4.15.0
webdriver-manager>=4.0.0
```

## Usage

### Compile a `.arweb` file

```bash
python main.py path/to/file.arweb
```

This lexes, parses, semantically checks, and generates HTML/CSS/JS into an `output/` directory, along with a merged Arabic error report (`output/report.txt`) if anything went wrong.

Options:

| Flag | Description |
|---|---|
| `-o`, `--output` | Output directory (default: `output`) |
| `--open` | Open the generated page in your default browser after compiling |
| `--capture-errors` | Load the generated page in headless Chrome and capture browser console errors |
| `--wait` | Seconds to wait after page load before capturing errors (default: `3`) |

Example with runtime error capture:

```bash
python main.py examples/todo.arweb --capture-errors --open
```

### Test the lexer standalone

Tokenizes a `.arweb` file and reports any unrecognized tokens, tagged by mode (HTML/CSS/JS/ATTR):

```bash
python test_lexer.py tests_5codes/sample.arweb
```

### Test the parser standalone

```bash
python test_parser.py path/to/file.arweb
```

### Run the test suite

```bash
python run_tests.py
```

Runs the compiler against the sample programs in `tests_5codes/`.

## How it works

1. **Lex & parse** — `ArabicHtmlLexer`/`ArabicHtmlParser` (ANTLR4) tokenize and parse the `.arweb` source. ANTLR's built-in error recovery keeps parsing past a syntax error, so every syntax error in the file is collected in one pass rather than stopping at the first one.
2. **Build AST** — `ArabicHtmlAstVisitor` walks the parse tree into an AST.
3. **Semantic analysis** — `SemanticAnalyzerVisitor` checks the AST (e.g. tag/type consistency) and collects any semantic errors.
4. **Code generation** — `CodeGeneratorVisitor` emits HTML, CSS, and JS files into the output directory.
5. **(Optional) Runtime capture** — the generated page is loaded in headless Chrome; browser console output and any WebDriver/terminal failures are captured, translated to Arabic, and mapped back to source locations using the generator's ID registry and JS source map.
6. **Report** — all errors collected across whichever phases ran (syntax, semantic, runtime) are merged into a single Arabic-language report, grouped by phase in pipeline order and sorted by line/column.

## Documentation

- `arabic_web_compiler_documentation.docx` — project documentation
- `arweb_compiler.pptx` — project presentation

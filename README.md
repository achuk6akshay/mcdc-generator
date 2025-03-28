Truth Table Generator

Overview

The Truth Table Generator is a JavaScript-based tool designed to parse logical expressions, generate corresponding truth tables, and evaluate conditions for Modified Condition/Decision Coverage (MC/DC) testing. The program is implemented using jQuery and JavaScript.

Features

Tokenization & Parsing: Converts logical expressions into an Abstract Syntax Tree (AST).

Truth Table Generation: Computes truth table combinations for Boolean variables.

MC/DC Evaluation: Identifies test cases that provide MC/DC coverage.

Interactive User Interface: Updates results dynamically based on user input.

Dependencies

jQuery

Usage

1. Input Expression

Enter a Boolean expression into the input field (e.g., A & B | C). The system will automatically process it.

2. Output Sections

Abstract Syntax Tree (AST): Displays the parsed structure of the expression.

Truth Table: Lists all possible truth value combinations for the given variables.

MC/DC Table: Highlights relevant test cases for achieving MC/DC coverage.

Main Functions

truthCombos(symbols)

Generates all possible truth table combinations for a set of Boolean symbols.

evalExpr(ast, bindings)

Evaluates a Boolean expression using an AST and a given set of variable bindings.

displayCombos(expression, symbols, ast, combos)

Renders the truth table for an expression in an HTML table format.

displayMcdc(expression, symbols, ast, combos)

Identifies and displays MC/DC test cases.

parse(tokens)

Parses tokenized input into an AST.

tokenize(str)

Tokenizes the input string into an array of symbols and operators.

Example Expression

Input:

(A & B) | C

Output:

Truth table for variables A, B, and C

Abstract Syntax Tree representation

MC/DC test cases

Debugging

Set DEBUG = true to enable console logging for debugging purposes.

License

MIT License

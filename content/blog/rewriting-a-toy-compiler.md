---
title: "Rewriting a Toy Compiler"
subtitle: "Java to Haskell (-75% LoC)"
notability: 6
tags: ["Refactoring", "PL", "FP"]
date: 2024-05-15T07:59:41-05:00
---

I've written a lot of [posts glorifying functional programming](/tags/fp) lately, but they tend to be abstract and hypothetical.
They roughly amount to "ADTs, higher-order-functions, and immutability are usually better[^better] than their counterparts".
In those posts, I attempted to *explain* why that is true; here I hope to *show* it.

[^better]: More concise, easy to write, easy to reason about, and easy to maintain.

## The Program

UW-Madison's [CS 536 (Compilers)](https://pages.cs.wisc.edu/~hasti/cs536/) features a[^a-project] mammoth project: a >4,300 LoC[^loc-def] compiler for a C-like language called "Base"[^language-name], written in Java (plus scanner and parser generators[^generators]).
As with most CS projects at UW, there was little flexibility in most of the implementation details (to enforce conformance to the auto-grader), so it was much more of a rigid puzzle (figure out what the professor/TAs want you to write, given comments and code skeletons) than a true programming challenge (figure out a solution and code structure that solves your problem, then make it maximally clean and maintainable).

[^a-project]: The program is spread out into 6 assignments (it isn't a single "project" in terms of the class), but that's unimportant when looking at the program as a whole.

[^loc-def]: LoC = Lines of Code.

[^language-name]: The language name and syntax (the superficial parts) change each semester, but the project itself (along with the rest of the course material) has been the same for at least a few years, if not decades (which isn't implausible, considering all the libraries it uses are from the 90s).

[^generators]: [jlex](https://www.cs.princeton.edu/~appel/modern/java/JLex/) (the scanner-generator) and [cup](https://www.cs.princeton.edu/~appel/modern/java/CUP/manual.html) (the parser-generator), which are roughly analogous to lex (flex) and yacc (bison), but in Java instead of C.

As you might expect, this can be quite frustrating for a student who is striving to learn about the essence of compilers.
Not only can the student not test their own understanding of the compilation process (since the process is already laid out in front of them in the skeleton code), but they also lose the freedom to structure their code in a way that is clear and intuitive for them[^clear].

[^clear]: Which might even be *better* than the provided way.

While writing the Java version, I frequently found myself frustrated about the code I was forced to write.
I kept thinking: *I could write this more clearly in Haskell in significantly fewer lines*.
So I did that, and the final Haskell line-count is 950, which is **less than 25%**[^loc] of the Java count.

[^loc]: Lines-of-Code is admittedly a deeply flawed metric (white-space, comments, imports, and differences in syntax are not accounted for), but the difference is *way* too big for this to be a false signal: there is clearly less logic in the Haskell version (or that logic is significantly more compact).

## The Comparison

- [(Full Haskell Code)](https://github.com/lucasscharenbroch/base-lang)
- (Java Code Excluded)[^no-java]

[^no-java]: I'm going to hold off on publicizing the full Java version; professors are usually uptight about "sharing code", and I can't imagine this project changes majorly from year to year.
The snippets I include directly in the post might be enough to give future students some hints, but not without picking up on some of my brilliant insights.

### File Structure

Here are the directory-listings (and file line counts) of both versions (just the source-code and build files).

(Java)
```
.
├── ast.java (2845)
├── base.cup (378)
├── base.jlex (339)
├── Codegen.java (254)
├── DuplicateSymNameException.java (2)
├── EmptySymTableException.java (2)
├── ErrMsg.java (36)
├── Makefile (69)
├── misc-test.base (49)
├── P6.java (74)
├── Sym.java (144)
├── SymTable.java (86)
└── Type.java (203)
```

(Haskell)
```
.
├── base-lang.cabal (23)
└── src
    ├── Ast.hs (67)
    ├── Generate.hs (358)
    ├── Lex.hs (55)
    ├── Main.hs (24)
    ├── Parse.hs (102)
    ├── Resolve.hs (216)
    └── Typecheck.hs (115)
```

Besides the lack of a source (and build[^build-dir]) directory and the high line counts in the Java version, the biggest difference is *the way program logic is split among files*.
The Haskell version is maximally straightforward: there is one file for data structures (`Ast.hs`), one for the main program (`Main.hs`), and one for each stage of compilation.
The same is not true for the Java version: instead of drawing file boundaries between *transformations of data*, files correspond (by necessity[^java-class]) with *objects* (data structures that are inseparably paired with associated functions).
Since a compiler primarily revolves around a single data structure (the Ast), this causes the majority of the program's logic to lie, intertwined, in a single file (`ast.java`).
The Java code is very monolithic as a result, which is ironic, considering OOP's claims to modularity.

[^build-dir]: I am very fond of separating binary and source files - the alternative is dumping them into the same folder, which is annoying with version control and a nightmare without it.

[^java-class]: Each file in java *must* contain at least one class that shares its name with the file.

### Data Structures

As noted above, the Haskell version has all of its shared data structures in a single file: `Ast.hs`.

```hs
module Ast where

import Text.Parsec (SourcePos)

type Error = String
type Id = String
type Body i t = ([Decl t], [Stmt i t])

data Decl t = Decl SourcePos (ValueType t) Id

data Location = Label String
              | LocalOffset Int
              | ParamOffset Int
              | LabelPlusOffset String Int
    deriving (Show)

addOffset :: Int -> Location -> Location
addOffset o (Label s) = LabelPlusOffset s o
addOffset o (LocalOffset o') = LocalOffset $ o + o'
addOffset o (ParamOffset o') = ParamOffset $ o + o'
addOffset o (LabelPlusOffset s o') = LabelPlusOffset s $ o + o'

-- i = id-associated data
-- t = type-size data (tuple types, function declarations)
type Ast i t = [TopDecl i t]
type UnresolvedAst = Ast () ()
type ResolvedAst = Ast (Type Int, Location) Int

data Type t = TVoid
          | TFn [ValueType t] (Type t)
          | TValType (ValueType t)
    deriving (Show, Eq)

data ValueType t = VTInteger
                 | VTLogical -- boolean
                 | VTString
                 | VTTuple Id t
    deriving (Show, Eq)

data TopDecl i t = FnDecl SourcePos (Type t) Id [Decl t] (Body i t) t
                 | TupleDef SourcePos Id [Decl t]
                 | Global (Decl t)

data Stmt i t = Inc SourcePos (Lvalue i)
              | Dec SourcePos (Lvalue i)
              | IfElse SourcePos (Expr i) (Body i t) (Maybe (Body i t))
              | While SourcePos (Expr i) (Body i t)
              | Read SourcePos (Lvalue i)
              | Write SourcePos (Expr i)
              | Return SourcePos (Maybe (Expr i))
              | ExprStmt SourcePos (Expr i) -- call, assignment

data Expr i = LogicalLit SourcePos Bool
            | IntLit SourcePos Int
            | StringLit SourcePos String
            | Assignment SourcePos (Lvalue i) (Expr i)
            | Call SourcePos (Lvalue i) [Expr i]
            | UnaryExpr SourcePos UnaryOp (Expr i)
            | BinaryExpr SourcePos BinaryOp (Expr i) (Expr i)
            | Lvalue SourcePos (Lvalue i)

data Lvalue i = Identifier SourcePos Id i
              | TupleAccess SourcePos (Lvalue i) Id i
    deriving (Show)

data UnaryOp = Negate | Not
data BinaryOp = Add | Sub | Mul | Div | Eq | Ne | Gt | Ge | Lt | Le | And | Or
```

The type variables **`i`** and **`t`** are necessary because they represent the information added to the Ast is during resolution (<b>`UnresolvedAst -> ResolvedAst`</b>).

Here's the Java version:

```java
abstract class ASTnode {
    protected void doIndent(PrintWriter p, int indent) { /* ... */ }
}

class ProgramNode extends ASTnode {
    public ProgramNode(DeclListNode L) { /* ... */ }
    public void nameAnalysis() { /* ... */ }
    public void typeCheck() { /* ... */ }
    public void codeGen() { /* ... */ } }
    public void unparse(PrintWriter p, int indent) { /* ... */ }

    private DeclListNode myDeclList;
    public static boolean noMain = true;
}

class DeclListNode extends ASTnode {
    public DeclListNode(List<DeclNode> S) { /* ... */ }
    public void nameAnalysis(SymTable symTab) { /* ... */ }
    public void nameAnalysis(SymTable symTab, SymTable globalTab) { /* ... */ }
    public void typeCheck() { /* ... */ }
    public void codeGen() { /* ... */ }
    public void unparse(PrintWriter p, int indent) { /* ... */ }

    private List<DeclNode> myDecls;
}

/* all remaining classses contain similar fields methods as those above */

class StmtListNode extends ASTnode { /* ... */ }
class ExpListNode extends ASTnode { /* ... */ }
class FormalsListNode extends ASTnode { /* ... */ }
class FctnBodyNode extends ASTnode { /* ... */ }
abstract class DeclNode extends ASTnode { /* ... */ }
class VarDeclNode extends DeclNode { /* ... */ }
class FctnDeclNode extends DeclNode { /* ... */ }
class FormalDeclNode extends DeclNode { /* ... */ }
class TupleDeclNode extends DeclNode { /* ... */ }
abstract class TypeNode extends ASTnode { /* ... */ }
class LogicalNode extends TypeNode { /* ... */ }
class IntegerNode extends TypeNode { /* ... */ }
class VoidNode extends TypeNode { /* ... */ }
class TupleNode extends TypeNode { /* ... */ }
abstract class StmtNode extends ASTnode { /* ... */ }
class AssignStmtNode extends StmtNode { /* ... */ }
class PostIncStmtNode extends StmtNode { /* ... */ }
class PostDecStmtNode extends StmtNode { /* ... */ }
class IfStmtNode extends StmtNode { /* ... */ }
class IfElseStmtNode extends StmtNode { /* ... */ }
class WhileStmtNode extends StmtNode { /* ... */ }
class ReadStmtNode extends StmtNode { /* ... */ }
class WriteStmtNode extends StmtNode { /* ... */ }
class CallStmtNode extends StmtNode { /* ... */ }
class ReturnStmtNode extends StmtNode { /* ... */ }
abstract class ExpNode extends ASTnode { /* ... */ }
class TrueNode extends ExpNode { /* ... */ }
class FalseNode extends ExpNode { /* ... */ }
class IdNode extends ExpNode { /* ... */ }
class IntLitNode extends ExpNode { /* ... */ }
class StrLitNode extends ExpNode { /* ... */ }
class TupleAccessNode extends ExpNode { /* ... */ }
class AssignExpNode extends ExpNode { /* ... */ }
class CallExpNode extends ExpNode { /* ... */ }
abstract class UnaryExpNode extends ExpNode { /* ... */ }
abstract class BinaryExpNode extends ExpNode { /* ... */ }
class NotNode extends UnaryExpNode { /* ... */ }
class UnaryMinusNode extends UnaryExpNode { /* ... */ }
abstract class ArithmeticExpNode extends BinaryExpNode { /* ... */ }
abstract class LogicalExpNode extends BinaryExpNode { /* ... */ }
abstract class EqualityExpNode extends BinaryExpNode { /* ... */ }
abstract class RelationalExpNode extends BinaryExpNode { /* ... */ }
class PlusNode extends ArithmeticExpNode { /* ... */ }
class MinusNode extends ArithmeticExpNode { /* ... */ }
class TimesNode extends ArithmeticExpNode { /* ... */ }
class DivideNode extends ArithmeticExpNode { /* ... */ }
class EqualsNode extends EqualityExpNode { /* ... */ }
class NotEqualsNode extends EqualityExpNode { /* ... */ }
class GreaterNode extends EqualityExpNode { /* ... */ }
class GreaterEqNode extends EqualityExpNode { /* ... */ }
class LessNode extends EqualityExpNode { /* ... */ }
class LessEqNode extends EqualityExpNode { /* ... */ }
class AndNode extends LogicalExpNode { /* ... */ }
class OrNode extends LogicalExpNode { /* ... */ }
```

The Java version is clearly trying to form a similar structure to the Haskell version, but by using subtyping instead of sum types (ADTs).
This is less natural for a plethora of reasons.

- With ADTs (the Haskell version), it's much easier to *see* the relationship between types (with the Java version, even when it's *significantly* compressed, you have to squint to read the "extends ..." clause, and visualize the hierarchy)
- Subtyping has much weaker invariants than ADTs
    - An <b>`ExpNode`</b> need not be something that directly extends it; it could also be anything that extends any of those extensions
    - It's difficult and unsafe (if not impossible) to de-structure (do a case-analysis and unwrap the associated data); this frequently leads to bad assumptions and unsafe casting
- "Classes" are not the natural way we think about every possible variant of an AST node
- Even with inheritance, there's significantly more code repetition required using this paradigm (this is the single biggest cause of the line-count difference); more code implies more places for bugs to emerge, and more difficulty to make global changes and maintain consistency

### Lexing

The Java version uses jlex, so I took the liberty to use the scanner-generator of my choice, and I chose `Text.Parsec.Tok`.
The library does most of the work here, and it's admittedly the pettiest way I avoided lines.


```hs
module Lex where

import Control.Monad
import Text.Parsec
import Text.Parsec.String
import Text.Parsec.Token as Tok

lexer :: Tok.TokenParser ()
lexer = makeTokenParser $ Tok.LanguageDef
    { commentStart = ""
    , commentEnd = ""
    , commentLine = "$"
    , nestedComments = False
    , identStart = letter <|> char '_'
    , identLetter = letter <|> digit <|> char '_'
    , opStart = oneOf ""
    , opLetter = oneOf "+-*/&|~<>="
    , reservedNames = ["void", "logical", "integer", "True", "False", "tuple",
                       "read", "write", "if", "else", "while", "return"]
    , reservedOpNames = []
    , caseSensitive = True
    }

identifier :: Parser String
identifier = Tok.identifier lexer

reserved :: String -> Parser ()
reserved = Tok.reserved lexer

reservedOp :: String -> Parser ()
reservedOp = Tok.reservedOp lexer

stringLiteral :: Parser String
stringLiteral = Tok.stringLiteral lexer

natural :: Parser Integer
natural = Tok.natural lexer

brackets :: Parser a -> Parser a
brackets = Tok.brackets lexer

parens :: Parser a -> Parser a
parens = Tok.parens lexer

dot :: Parser ()
dot = void $ Tok.dot lexer

colon :: Parser ()
colon = void $ Tok.colon lexer

commaSep :: Parser a -> Parser [a]
commaSep = Tok.commaSep lexer

symbol :: String -> Parser ()
symbol = void . Tok.symbol lexer
```

That being said, the Java version (`base.jlex`) isn't exactly flattering with its mixed use of (more) clumsy classes (especially CharNum[^tried-refactor]) and funny scanner-generator syntax.

[^tried-refactor]: The lexer was the second (of six) assignments, and it was early in the semester before my hope was shattered: I tried to refactor <b>`CharNum`</b> out, to no avail (the jlex program combined with the autograder barred me from accessing variables in the underlying class).

```
class TokenVal {
    int lineNum;
    int charNum;

    TokenVal(int lineNum, int charNum) {
        this.lineNum = lineNum;
        this.charNum = charNum;
    }
}

class IntLitTokenVal extends TokenVal { /* ... */ }

class IdTokenVal extends TokenVal { /* ... */ }

class StrLitTokenVal extends TokenVal { /* ... */ }

// The following class is used to keep track of the character number at 
// which the current token starts on its line.
class CharNum {
    static int num = 1;
}
%%

DIGIT=        [0-9]
WHITESPACE=   [\040\t]
LETTER=       [a-zA-Z]
ESCAPEDCHAR=  [nst'\"\\]

NOTNEWLINEORESCAPEDCHAR=   [^\nnt'\"?\\]
NOTNEWLINEORQUOTE= [^\n\"]
NOTNEWLINEORQUOTEORESCAPE= [^\n\"\\]

%implements java_cup.runtime.Scanner
%function next_token
%type java_cup.runtime.Symbol

%eofval{
return new Symbol(sym.EOF);
%eofval}

%line

%%

"void"    { Symbol S = new Symbol(sym.VOID, new TokenVal(yyline+1, CharNum.num));
            CharNum.num += yytext().length();
            return S;
          }
		  
"logical"    { Symbol S = new Symbol(sym.LOGICAL, new TokenVal(yyline+1, CharNum.num));
            CharNum.num += yytext().length();
            return S;
          }
		  
"integer"    { Symbol S = new Symbol(sym.INTEGER, new TokenVal(yyline+1, CharNum.num));
            CharNum.num += yytext().length();
            return S;
          }
		  
"True"    { Symbol S = new Symbol(sym.TRUE, new TokenVal(yyline+1, CharNum.num));
            CharNum.num += yytext().length();
            return S;
          }

...
```

### Parsing

Parsing is one of Haskell's strong-suits, and Parsec is a godsend.
Hilariously, the Haskell code for parsing is even shorter than the BNF provided by the course.

```
[lucas@arch base-lang]$ wc -l base.grammar src/Parse.hs
 112 base.grammar
 102 src/Parse.hs
```

The java version uses cup (similar to yacc), which has the same flaws as the lexer.

### Name Analysis (Resolution)

This part was actually pretty similar between the two versions.
The Java version is a little odd in passing around a reference to a <b>`SymTable`</b> instead of just using a global variable (and also using the same table for variables and tuple-typedefs), but other than that, they do most of the same things, just in slightly different ways.

### Type Checking

Type checking is also pretty much the same; each node recursively calls the type-checking function on its children, potentially returning the resulting type (if there is one), or throwing an error if they mismatch.

Haskell (of course) uses ADTs for errors (the [`Either`](https://hackage.haskell.org/package/base-4.20.0.0/docs/Data-Either.html) type) instead of try-catch and global state, and I think that many of the advantages of this have already been widely discussed[^rust], but here's another (less obvious) one.

[^rust]: This feature is discussed because it's used in several popular modern languages (i.e. Go and Rust).

It's easier to forget to do an error-throwing operation in a void function then it is to do the same in an error-type-returning function.
For example, in the Java version:

```java
class PostIncStmtNode extends StmtNode {
    /* ... */

    public void typeCheck(Type retType) {
        /* Oops! I commented out myExp.typeCheck()

        Type type = myExp.typeCheck();

        if (!type.isErrorType() && !type.isIntegerType()) {
            ErrMsg.fatal(myExp.lineNum(), myExp.charNum(),
                         "Arithmetic operator used with non-integer operand");
        }

        */
    }

    /* ... */
}
```

Consider the <b>`typeCheck`</b> function, which returns <b>`void`</b>.
Even if I comment out the code that might throw an error (<b>`myExp.typeCheck()`</b>), the code still compiles and runs, it just silently skips checking the type of any <b>`PostIncStmtNode`</b>.

The Haskell version, on the other hand, requires exactly one return value, of type <b>`CheckM ()`</b> (<b>`Either Error ()`</b>).
In order to ignore the recursive-check, you would have to explicitly type "`return ()`"; there's no way to leave the body blank without compilation failing.

```hs
checkStmt :: Type T -> Stmt R T -> CheckM ()
checkStmt _ (Inc pos lval) = checkLvalue lval >>= expectType_ pos (TValType VTInteger)
-- ...
```

This may seem minor, but even small things like this can add up to make big assurances when a program compiles.

### Code Generation

In the Java version, all code generation functions are void: all generation happens through a global variable of type <b>`Codegen`</b>, which is effectively a global string that can be appended to plus some helper functions to conveniently append to it.
There's also a means of generating unique labels.

```java
public class Codegen {
    // file into which generated code is written
    public static PrintWriter p = null;

    public static final String FP = "$fp";
    public static final String SP = "$sp";
    public static final String RA = "$ra";
    public static final String V0 = "$v0";
    public static final String V1 = "$v1";
    public static final String A0 = "$a0";
    public static final String T0 = "$t0";
    public static final String T1 = "$t1";

    private static final int MAXLEN = 4; // for pretty printing generated code
    private static int currLabel = 0; // for generating labels
    public static void generateWithComment(String opcode, String comment,
                                        String arg1, String arg2, String arg3) {
        int space = MAXLEN - opcode.length() + 2;

        p.print("\t" + opcode);
        if (arg1 != "") {
            for (int k = 1; k <= space; k++) 
                p.print(" ");
            p.print(arg1);
            if (arg2 != "") {
                p.print(", " + arg2);
                if (arg3 != "") 
                    p.print(", " + arg3);
            }
        }
        if (comment != "") 
            p.print("\t\t# " + comment);
        p.println();
    }

    public static void generateWithComment(String opcode, String comment, String arg1, String arg2) { /* ... */ }
    public static void generateWithComment(String opcode, String comment, String arg1) { /* ... */ }
    public static void generateWithComment(String opcode, String comment) { /* ... */ }
    public static void generate(String opcode, String arg1, String arg2, String arg3) { /* ... */ }
    public static void generate(String opcode, String arg1, String arg2) { /* ... */ }
    public static void generate(String opcode, String arg1) { /* ... */ }
    public static void generate(String opcode) { /* ... */ }
    public static void generate(String opcode, String arg1, String arg2, int arg3) { /* ... */ }
    public static void generate(String opcode, String arg1, int arg2) { /* ... */ }
    public static void generateIndexed(String opcode, String arg1, String base, int offset, String comment) { /* ... */ }
    public static void generateIndexed(String opcode, String arg1, String base, int offset) { /* ... */ }
    public static void generateLabeled(String label, String opcode, String comment, String arg1) { /* ... */ }
    public static void generateLabeled(String label, String opcode, String comment) { /* ... */ }
    public static void genPush(String s) { /* ... */ }
    public static void genPop(String s) { /* ... */ }
    public static void genLabel(String label, String comment) { /* ... */ }
    public static void genLabel(String label) { /* ... */ }
    public static String nextLabel() { /* ... */ }
}
```

The Haskell version is kind of similar; it also provides a means of appending data directives (but not instructions), and generating unique labels.

```hs
data GenState = GenState
    { labelStream :: [Label]
    , dataDirectives :: [DataDirective]
    }

initialGenState :: GenState
initialGenState = GenState
    { labelStream = map (("L"++) . show) [0..]
    , dataDirectives = []
    }

type GenM = State GenState

freshLabel :: GenM Label
freshLabel = (head . labelStream <$> get) <* modify (\s -> s { labelStream = tail . labelStream $ s })

addData :: [DataDirective] -> GenM ()
addData dds = modify (\s -> s { dataDirectives = dataDirectives s ++ dds })
```

Instructions are treated as the return value from generation functions (rather than a global state).

```hs
generate :: ResolvedAst -> MipsProgram
generate ast = MipsProgram (concat instructionss) (dataDirectives resState)
    where (instructionss, resState) = runState (mapM genTopDecl ast) initialGenState


genTopDecl :: TopDecl R T -> GenM [Instruction]
genGlobal :: Decl T -> GenM ()
genBody :: Label -> Body R T -> GenM [Instruction]
genStmt :: Label -> Stmt R T -> GenM [Instruction]
genExpr :: Expr R -> GenM [Instruction]
-- ...
```

Instructions are not strings, but rather hard-coded variants of the <b>`Instruction`</b> type, one per opcode.
They can be converted to strings via <b>`show`</b>.

```hs
data Register = T0 | T1 | V0 | RA | SP | FP | A0

instance Show Register where
    show T0 = "$t0"
    show T1 = "$t1"
    show V0 = "$v0"
    show RA = "$ra"
    show SP = "$sp"
    show FP = "$fp"
    show A0 = "$a0"

data Instruction = TextLabel Label
                 | MainFnLabel
                 | Comment String
                 | Commented Instruction String
                 | StoreIdx Register Int Register -- sw $ra, 0($sp)
                 | LoadIdx Register Int Register -- lw $ra, 0($fp)
                 | StoreLabel Register Label -- sw $t0, _x
                 | LoadLabel Register Label -- lw $t0, _x
                 | SubUnsigned Register Register Int -- subu $sp, $sp, 4
                 | AddUnsigned Register Register Int -- addu $sp, $sp, 4
                 | Move Register Register -- move $t0, $t1
                 | Jump Register -- jr $ra
                 | JumpLabel Label -- j _L0
                 | Call Register -- jalr $t0
                 | LoadImm Register Int -- li $v0, 1
                 -- ...

instance Show Instruction where
    show (TextLabel l) = l ++ ":"
    show MainFnLabel = ".globl main\nmain:"
    show (Comment c) = "# " ++ c
    show (Commented i c) = show i ++ "# " ++ c
    show (StoreIdx r0 offset r1) = "sw " ++ show r0 ++ ", " ++ show offset ++ "(" ++ show r1 ++ ")"
    show (LoadIdx r0 offset r1) = "lw " ++ show r0 ++ ", " ++ show offset ++ "(" ++ show r1 ++ ")"
    show (StoreLabel r l) = "sw " ++ show r ++ ", " ++ l
    show (LoadLabel r l) = "lw " ++ show r ++ ", " ++ l
    show (SubUnsigned r0 r1 i) = "subu " ++ show r0 ++ ", " ++ show r1 ++ ", " ++ show i
    show (AddUnsigned r0 r1 i) = "addu " ++ show r0 ++ ", " ++ show r1 ++ ", " ++ show i
    show (Move r0 r1) = "move " ++ show r0 ++ ", " ++ show r1
    show (Jump r) = "jr " ++ show r
    show (JumpLabel l) = "j " ++ l
    show (Generate.Call r) = "jalr " ++ show r
    show (LoadImm r i) = "li " ++ show r ++ ", " ++ show i
```

Using ADTs instead of strings prevents the programmer from writing non-existent or ill-formed instructions, it also makes programming instructions easier (because of auto-complete and in-IDE linting).
For example, here's both versions of the generation for while-loops.

Java:
```java
class WhileStmtNode extends StmtNode {
    /* ... */

    public void codeGen(String retLabel) {
        String cond = Codegen.nextLabel();
        String done = Codegen.nextLabel();

        Codegen.genLabel(cond, "cond");
        myExp.vCodeGen();
        Codegen.genPop(Codegen.T0); // pop off condition
        Codegen.generate("beqz", Codegen.T0, done); // branch == 0
        myStmtList.codeGen(retLabel);
        Codegen.generate("j", cond);
        Codegen.genLabel(done, "done");
    }

    /* ... */
}
```
(Note that all arguments of all generation functions used above are strings.)

Haskell:

```hs
genStmt retLabel (While _pos cond body) = do
    beginLabel <- freshLabel
    doneLabel <- freshLabel
    generatedBody <- genBody retLabel body
    return [TextLabel beginLabel] +>+
           genExpr cond +&+
           [Pop T0, BranchEqZ T0 doneLabel] ++
           generatedBody ++ [JumpLabel beginLabel] ++
           [TextLabel doneLabel]
```

<b>`Instruction`</b>s are unequivocally more useful than strings.
They can even serve as an IR: I could have added an optimization step after generation, that looked something like this:

```hs
optimize :: [Instruction] -> [Instruction]
optimize (Push r):(Pop r'):rest
    | r == r' = optimize rest
optimize (Pop r):(Push r'):rest
    | r == r' = optimize rest
optimize x:xs = optimize xs
optimize x = x
```

This would actually be really important if I wanted to make this compiler useful[^useful].

[^useful]: Which, for the record, is not my intention (see the [github readme](https://github.com/lucasscharenbroch/base-lang/tree/main?tab=readme-ov-file#code-generation) for more on that).

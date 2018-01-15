---
layout: post
heading: Making Nashorn interruptible 
description: Rewriting Nashorn AST - Making Nashorn interruptible 
author: nick
---

Nashorn is a JavaScript Engine on the JVM that shipped with Java 8, it replaces the slow and outdated Rhino engine and is built using some new bytecode primitives that shipped with Java 8 JVM, namely invokeDynamic. Nashorn, unlike Rhino is actually pretty quick, the reason for this is that Nashorn interprets JavaScript and then compiles that into JVM bytecode that then runs like normal Java code with all the magic JVM optismisation voodoo that comes with it (not really but pretty close). This is why we use Nashorn in Sandbox to execute JavaScript and allow our users to write custom logic for their stubs, it does pose a problem though for responsibly executing that JavaScript in a multi-tenant environment.

Interpreting JS into bytecode is great for performance, but it makes executing code quite risky. In Rhino there were a few options to get more control over and effect an in-progress execution, the simplest is the `observeInstructionCount` hook which is triggered for every Nth instruction executed. Using this hook allowed you to run JavaScript and then get a callback every N instructions that meant you could potentially terminate execution if a limit had been exceeded. In Nashorn this does not exist, and is in fact a common problem to most interpreted languages running on the JVM, JRuby for example has the same problem. Because the code running on the JVM isn't aware of the JVM threading behaviour, it doesn't check for and honour calls to `Thread.interrupt()`, which is the standard Java primitive for outside influence over an executing thread, this effectively newerts our ability to control a process once started. Again, bad for a multi-tenant environment executing arbitrary code.

Since a Sandbox executes arbitrary JS and for most pricing tiers is in a multi-tenant environment, the fact that Nashorn doesn't support this wasn't acceptable, we wanted Nashorn's speed ES5/ES2016 language support but needed to be able to execute code safely. We managed to achieve it (quite a while ago now, but took us a while to write this up) with a few simple tricks.

### Nashorn Interpreting
When you evaluate JavaScript using the Nashorn engine it processes it in two main stages, first it interprets the JS into an AST (Abstract Syntax Tree) that matches the JS language. It is at this first stage where syntactic errors are picked up and thrown as errors (because it would be impossible to represent in the AST, as its invalid JS!). The AST consists of a tree of Nodes all extending from the `jdk.nashorn.internal.ir.Node` class. There are quite a few subclasses, but less than you would expect for a set of types that represent the entire JS language:

```
Node (jdk.nashorn.internal.ir)
Statement (jdk.nashorn.internal.ir)
ExpressionStatement (jdk.nashorn.internal.ir)
ThrowNode (jdk.nashorn.internal.ir)
BlockStatement (jdk.nashorn.internal.ir)
VarNode (jdk.nashorn.internal.ir)
CatchNode (jdk.nashorn.internal.ir)
EmptyNode (jdk.nashorn.internal.ir)
IfNode (jdk.nashorn.internal.ir)
SplitReturn (jdk.nashorn.internal.ir)
SetSplitState (jdk.nashorn.internal.ir)
LexicalContextStatement (jdk.nashorn.internal.ir)
TryNode (jdk.nashorn.internal.ir)
SplitNode (jdk.nashorn.internal.ir)
LabelNode (jdk.nashorn.internal.ir)
WithNode (jdk.nashorn.internal.ir)
BreakableStatement (jdk.nashorn.internal.ir)
LoopNode (jdk.nashorn.internal.ir)
WhileNode (jdk.nashorn.internal.ir)
ForNode (jdk.nashorn.internal.ir)
SwitchNode (jdk.nashorn.internal.ir)
ReturnNode (jdk.nashorn.internal.ir)
JumpStatement (jdk.nashorn.internal.ir)
BreakNode (jdk.nashorn.internal.ir)
ContinueNode (jdk.nashorn.internal.ir)
JumpToInlinedFinally (jdk.nashorn.internal.ir)
Block (jdk.nashorn.internal.ir)
PropertyNode (jdk.nashorn.internal.ir)
Expression (jdk.nashorn.internal.ir)
UnaryNode (jdk.nashorn.internal.ir)
GetSplitState (jdk.nashorn.internal.ir)
IdentNode (jdk.nashorn.internal.ir)
LiteralNode (jdk.nashorn.internal.ir)
LexerTokenLiteralNode in LiteralNode (jdk.nashorn.internal.ir)
PrimitiveLiteralNode in LiteralNode (jdk.nashorn.internal.ir)
ArrayLiteralNode in LiteralNode (jdk.nashorn.internal.ir)
BinaryNode (jdk.nashorn.internal.ir)
BaseNode (jdk.nashorn.internal.ir)
IndexNode (jdk.nashorn.internal.ir)
AccessNode (jdk.nashorn.internal.ir)
ObjectNode (jdk.nashorn.internal.ir)
JoinPredecessorExpression (jdk.nashorn.internal.ir)
TypeHolderExpression in LocalVariableTypesCalculator (jdk.nashorn.internal.codegen)
LexicalContextExpression (jdk.nashorn.internal.ir)
FunctionNode (jdk.nashorn.internal.ir)
CallNode (jdk.nashorn.internal.ir)
TernaryNode (jdk.nashorn.internal.ir)
RuntimeNode (jdk.nashorn.internal.ir)
CaseNode (jdk.nashorn.internal.ir)
```

The AST is constructed whenever new JS is evaluated, either from Java or from within JavaScript via `eval()`. The work is done in the aptly named `jdk.nashorn.internal.parser.Parser` class in the `parse(...)` method. This method takes a String (your JS code) and returns a FunctionNode tree (the AST). 

### Nashorn Compiling

The second main stage is to take the AST `FunctionNode` tree and compile it into bytecode to be executed. This is done by the `jdk.nashorn.internal.codegen.Compiler` class in the `compile(...)` method. The `compile(...)` method has a number of predefined stages, defined in the  `jdk.nashorn.internal.codegen.Compiler.CompilationPhases` class. The first phase is called the `ConstantFoldingPhase`, we don't actually know what this phase does but to achieve our goal of gaining the ability to interrupt executing Nashorn bytecode, we don't really care.

### Rewriting AST

Ok so we now have a simple understanding of how some JS code makes its way from a String to some executing bytecode, but how do we add the ability to interrupt it when Nashorn doesn't support it? We need to hijack the AST parsing defined above to allow us to customise the AST `FunctionNode` tree before it gets passed to the `Compiler`, that way we can add extra code to trigger a function controlled by us that does honour a `Thread.interrupt()` so that when a run-away process is timed out, it actually stops. This is slightly tricker than it seems, and is possibly not acceptable to you, depending on the constraints for your implementation. We are going to manipulate some JDK classes.

As the Nashorn Parser and Compiler classes were obviously not intended to allow outside changes to the AST, they do not expose the proper hooks to allow access to the objects we require, nor will they in the future i'd imagine since all of this lives in the `jvm.nashorn.internal` package which I assume to mean, hands off. So we will need to do some bytecode manipulation of the JDK classes while they are being classloaded.

There are lots of ways of manipulating bytecode which I won't go into here, but the best choice for us weighing up future breaking changes in the JDK, packaging, testing and avoiding runtime failures was to write a simple Java PreMain agent, [more details](https://zeroturnaround.com/rebellabs/how-to-inspect-classes-in-your-jvm/). Doing it this way meant we weren't permanently changing JDK classes and could choose to selectively rewrite classes if they matched our expected method signatures etc, and fall back to standard JDK behaviour if they don't, rather than maybe exploding.

So we want to hijack the JDK classes to give us a hook to rewrite the AST nodes between `Parser` and `Compiler`, after much wading through code considering which classes and methods are easiest to manipulate (changing final / inner / static classes is tough or impossible) the easiest place is in:
```java
jdk.nashorn.internal.codegen.CompilationPhase$ConstantFoldingPhase.transform()
```
Which is the first compilation phase that is called by the `Compiler`, so we aren't really getting a hook before the compiler runs, but close enough. I used [ByteBuddy](http://bytebuddy.net/) to do the bytecode rewriting, but you could use any of its alternatives i'd assume.

The simple PreMain:
```java
import net.bytebuddy.agent.builder.AgentBuilder;
import net.bytebuddy.asm.Advice;
import java.lang.instrument.Instrumentation;
import static net.bytebuddy.matcher.ElementMatchers.named;

public class PreMain {
    public static void premain(String agentArgs, Instrumentation instrumentation) {
        new AgentBuilder.Default()
                .type(named("jdk.nashorn.internal.codegen.CompilationPhase$ConstantFoldingPhase"))
                .transform(
                    (builder, typeDescription, classLoader, module) -> {
                        return builder.method(named("transform")).intercept(Advice.to(CompilerAdviser.class));
                    }
                ).installOn(instrumentation);
    }
}
```

and the `CompilerAdvisor.class`

```java
import jdk.nashorn.internal.ir.FunctionNode;
import net.bytebuddy.asm.Advice;

import java.util.function.Function;

import static net.bytebuddy.implementation.bytecode.assign.Assigner.Typing.DYNAMIC;

public class CompilerAdviser {

    @Advice.OnMethodExit
    public static void onExit(@Advice.Return(readOnly = false, typing = DYNAMIC) Object returned) {
        if(returned instanceof FunctionNode){
            FunctionNode result = (FunctionNode) returned;
            // rewrite AST! do stuff here!
            returned = result;
        }

    }

}
```

So using this hook we can rewrite any AST before it gets compiled. But what are we going to rewrite? The simplest runaway execution to defend against is infinite loops like `while(true) { //do stuff }`. The best way to walk the AST and rewrite it is the use the out of the box `jdk.nashorn.internal.ir.visitor.SimpleNodeVisitor` class, and overriding the `leaveWhileNode` method which is called for each AST Node that represents a `while(){ ... }` statement. The `WhileNode` has a `body` property that is an AST node that represents the `{ ... }` portion of the while statement. We want to wrap the `body` node within a new `IfNode` so that with each iteration of the while loop, our if() statement is called, effectively creating a `while(true) { if(ourFunction()){ ... } }`.

```java
    @Override
    public Node leaveWhileNode(WhileNode whileNode) {

        IfNode ifContinue = new IfNode(
                whileNode.getLineNumber(),
                whileNode.getToken(),
                whileNode.getFinish(),
                new CallNode(
                        whileNode.getLineNumber(),
                        whileNode.getToken(),
                        whileNode.getFinish(),
                        new IdentNode(
                                whileNode.getToken(),
                                whileNode.getFinish(),
                                "ourFunction"
                        ),
                        new ArrayList(),
                        false
                ),
                whileNode.getBody(),
                null
        );

        Block ifBlock = new Block(
                whileNode.getToken(),
                whileNode.getFinish(),
                new Statement[]{
                        ifContinue
                }
        );
        return whileNode.setBody(this.getLexicalContext(), ifBlock);
    }
```

tl;dr; You can ignore most of it other than the `ourFunction` in the `IdentNode`, that is the name of the function that we are going to inject. So now the compiled bytecode will call `ourFunction()` for every iteration of any while loop, woot! 

So to wrap it up we just need to define a function to check if we need to interrupt the current execution, that function might look like this (you don't have to use `isInterrupted()` any flag to drive the condition off will do):

```js

function ourFunction(){
    if(Thread.currentThread().isInterrupted()){
        throw new RuntimeException("Interrupted!");
    }
    return true;
}
```

And then run our `PreMain` bytecode rewriter using the `-javaagent:...` argument.
```
java -javaagent:ast-rewrite.jar ...`
```

Job done! We can now interrupt execution for infinite while loops. Obviously there a quite a number of other possible JavaScript incantations that will cause high resource consumption, but with access to the AST and an ability to manipulate it the hard work is done we just need to extend our rewriting to cover a few more cases, a future blog post possibly.




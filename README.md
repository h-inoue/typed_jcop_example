# typed_jcop_example

Typed JCop is a prototype language that supports typechecking several
JCop features.  It is also a implementation of ContextFJ_{<:}, which
is a calculus supporting inheritance of layer inheritance, subtyping
of layer types, first-class layers and layer swapping.


How to get compiler
------
Go to https://github.com/h-inoue/JCop/releases and download jcop.jar.


Language Constructs
------
### Layer Definition ###
TJCop supports only layer declaration as class-in-layer style.

    [swappable] layer L [extends L1] [requires L2, L3]{
        partial method declarations...
    }


### Partial Methods Declaration ###

A name of partial methods must include full qualified class name.  You
can also include proceed() call in the partial method body..

    public void pckg.C.m( args ){
        ....
    }

### Layer Activation/Swapping ###

    with( expression ){
        ....
    }

    swap( LayerName, expression ){
        ....
    }


How to use the compiler
-------

To compile, you can use following command.

    java -jar -ea "tjcop.jar" <TJCop specific compiler options> class

In bash, you can define following code as shell script, setting the
variable $JCOP_HOME.

    \#!/bin/bash
    java -jar -ea "$JCOP_HOME/jcop.jar" $*

You can compile the example as follows.
    
    cd transfersystem
    jcopc.sh -d bin -sourcepath src main.Main

Then, you can run a program by normal java command.

    java -cp bin main.Main

Type  Checking Examples
------
A distinct feature of Typed JCop is COP originated type checking.  For
example, considering to compile following code.

### Simple Example ###

    /** Weather.jcop **/
    package main;
    public layer Weather{
	    public String main.Main.getWeather(){
		    return "not implemented";
	    }
    }

    /** Main.jcop **/
    package main;
    public class Main{
        public static void main(String[] args) {
		    String str = new Main().getWeather();
		    System.out.println(str);
	    }
    }

Since the method getWeather() is newly declared in a layer Weather,
the method call will cause following compile error.

    $ jcopc.sh Main.jcop
    CopCompileError: [/Users/hiro/Dropbox/work/src/jcop/simple1/./Main.jcop:9:
        Semantic Error: There is no valid method getWeather() in this context [].]

If we modify the sourcecode to activate an instance of Weather layer,
it works well.

    /** Main.jcop **/
    package main;
    public class Main{
        public static void main(String[] args) {
		    with(new Weather()){
			    String str = new Main().getWeather();
			    System.out.println(str);
            }
        }
    }

If a program pass through COP specific type checking, following
message appears.

    $ jcopc.sh Main.jcop
    [COP type check completed :)]

### Example of swapping ###

Here, let's consider an example of bank transfer system.  

    /** EncryptionLayer.jcop **/
    package encryption;
    import account.*;
    public swappable layer EncryptionLayer {
        public float account.Account.getBalance() {
		    return Encryption.encrypt(proceed());
	    }
    }

Suppose that we want to deactivate the layer temporally.  We can
define such a feature by declaring a sublayer of EncryptionLayer.

    /** TemporallyDecryptionLayer.jcop **/
    package encryption;
    public layer TemporallyDecryptionLayer extends EncryptionLayer{
	    public float account.Account.getBalance() {
		    return proceed();
	    }
    }

    swap(EncryptionLayer, tmplayer){
        /* temporally deactivate encryption */
    }

Whole example sourcecode is in this repository.


Current Limitation
------
Current Typed JCop is very restricted language, compared with original
JCop.  Following features of original JCop would cause a error or an
inconsistency of type checking.

- Activating layer list.
- Deactivating layer.(without)
- Layer declaration as layer-in-class style
- before/after modifier
- staticactive layer
- Layer local field and method
- Calling superproceed()
- Using thislayer
- Declarative layer composition
- Reflective layer composition
- JCop API features

### Bug ###
- When proceed() call continues to find a method of super class, it
  will stack.

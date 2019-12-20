# Androlic
Developers can take Androlic as a command line tool or develop self-defined static analysis tools by extending APIs of Androlic. Whatever we want to do, we need to get the following file:

* androlic.jar

## Command Line Tool
If we take Androlic as a command line tool, we can use it through following command:

    java -jar androlic.jar [options]

The command line options of Androlic are shown as following:
    
    -apkName	    The name of apk under analysis. For example, douyin.apk.
    -javaHome       The directory where rt.jar lies. For example, C:\\java\\lib\\rt.jar.
    -apkBasePath	    The directory where apk file lies. For example, D:\\test.
    -outputBasePath     The directory where the output file of Androlic lies. 
    -androidPath	    The directory where android.jar lies.
    -debugMode          Indicate whether Androlic is in debug mode. Debug mode will output detailed path message which is time consuming. 1 means yes, 0 means no. 0 by default.
    -isJimpleOutput	    Indicate whether Androlic output jimple file or not. 1 means yes, 0 means no. 1 by default.
    -maxRunningTime	    The maximum running time of Androlic. 1800000 milliseconds (30 minutes) by default.
    -maxLoopUnrollNumber      The maximum unroll time when dealing with loop structure. 1 by default.
    -maxPathNumber      The maximum path number during analysis. 40000 by default.
    -maxRecursiveInvocationLevel      The maximum number of recursive invocation. 0 by default.
    -entryMethod      entry method of analysis. dummy main method of Activity by default. If you want to set it, please input the complete method name. For example, com.test.testMethod
    -initMethodSubsignature       If entry method is a non-static method, we need to indicate the init method of object that entry method belongs to. Non-parametric init method by default. The format should be subsignature in jimple. For example, void <init>(int).
    -configureFile      The path of configure file. For example, D:\\configuration.txt

Note that -apkName, -javaHome, -apkBasePath, -outputBasePath and -androidPath are compulsory options, which means they must be set. For example:

    java -jar androlic.jar –apkName douyin.apk -javaHome C:\\Program Files\\Java\\rt.jar –apkBasePath D:\\test –outputBasePath D:\\apkPath –androidPath D:\\android-platform
    
If we set -configureFile, then the configuration item in the configure file will cover the configure item set in command line. An example  of configure file is shown as following:
    
    apkName  BasicTypeTest.apk
    apkBasePath  test-apk
    outputBasePath  jimple\\test-apk
    isJimpleOutput  1
    androidPath  D:\\download\\soot-path\\android-platforms-master
    maxRunningTime
    maxLoopUnrollNumber
    maxPathNumber
    maxRecursiveInvocationLevel
    entryMethod com.example.basictypetest.MainActivity.testCastExpr
    initMethodSubsignature 
    com.example.arraytest.MainActivity.testArrayParam

Each line in the file denotes a configuration. Configuration item and its value are separated by blank space. If configuration item is illegal or the value is null, then the configuration won't take effect. In the above file, only the first five lines will take effect.

We can make configuration in file through the following command:

    java -jar androlic.jar -configureFile D:\\configuration.txt

## Extend Androlic

Androlic provides several APIs through which developer can carry out custom analysis tasks. These APIs are listed below:
1. androlic.execution.ISymbolicEngineInstrumenter provides three methods to instrument symbolic engine:
    ```
    // Execute before currentUnit is processed by symbolic engine
    public void onPreStmtExecution(Unit currentUnit, GlobalMessage globalMessage);
    // Execute after currentUnit is processed by symbolic engine
    public void onPostStmtExecution(Unit currentUnit, GlobalMessage globalMessage);
    // Execute if androlic-defined exception occurs
    public void onExceptionProcess(Unit currentUnit, GlobalMessage globalMessage, AbstractAndrolicException exception);
    ```

2. androlic.execution.processor.library.proxy.ILibraryInvocationProcessor provides two methods to process library method:
    ```
    // InvokeExpr of library method is the right operand of AssignStmt, we need to return the value of InvokeExpr. If we can't process the libraryInvokeExpr, please return null
    public IBasicValue getLibraryInvocationReturnValue(AssignStmt stmt, InvokeExpr libraryInvokeExpr, 
          ContextMessage context, GlobalMessage globalMessage);
    // InvokeExpr of library method is contained in InvokeStmt. Return false if we can't process the libraryInvokeExpr
    public boolean processLibraryInvocation(InvokeStmt stmt, InvokeExpr libraryInvokeExpr, 
          ContextMessage context, GlobalMessage globalMessage)
    ```

3. androlic.execution.processor.rightop.expr.newex.INewExprProcessor provides a method to process NewExpr:
    ```
    // Return self-defined object which extends androlic.entity.value.heap.ref.NewRefObject
    public NewRefHeapObject getNewHeapObject(AssignStmt stmt, NewExpr newExpr,ContextMessage context, GlobalMessage globalMessage);
    ```

4. androlic.execution.processor.rightop.expr.newex.INewArrayExprProcessor provides a method to process NewArrayExpr:
    ```
    // Return self-defined array object which extends androlic.entity.value.heap.array.NewArrayHeapObject
    public NewArrayHeapObject getNewArrayHeapObject(AssignStmt stmt, NewArrayExpr newArrayExpr, 
          ContextMessage context, GlobalMessage globalMessage);
    ```
5. androlic.execution.processor.interprocedure.IMethodInterProceduralJudge provides a method to deal with inter-procedural analysis: 
    ```
    // Return true if the InvokeExpr ie needs to be unfolded to perform inter-procedural analysis. Otherwise, return false
    public boolean isNeedInterProceduralAnalysis(InvokeExpr ie, ContextMessage context, GlobalMessage globalMessage);
    ```




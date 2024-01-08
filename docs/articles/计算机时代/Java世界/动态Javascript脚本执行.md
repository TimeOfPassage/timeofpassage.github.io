# #动态Javascript脚本执行

## 脚本准备

```javascript
var Jutil = Java.type("com.api.common.JsInvokeHelper");
function test() {
    function test3() {
        print(ctx);
    }

    test3();
    // print(ctx);  // RespDTO(resultCode=200, message=Ok, data=null)
    // print(typeof ctx); // object
    print('resultCode: ', ctx.resultCode)
    print("js 执行输出");

    print("js 执行Java Method输出--->", Jutil.testPrint("js"));
    return "hello javascript";
}
```

## Core处理

pom沙箱处理

```xml
<dependency>
    <groupId>org.javadelight</groupId>
    <artifactId>delight-nashorn-sandbox</artifactId>
    <version>0.2.5</version>
</dependency>
```

核心处理

```java
public class JsUtils {

    private static final JsUtils INSTANCE = new JsUtils();

    private JsUtils() {
    }

    private        NashornSandbox sandbox;
    private static ScriptEngine   engine;

    public static JsUtils getInstance() {
        return INSTANCE;
    }

    public void createJavaScriptEngineIfNotExist() {
        if (engine == null) {
            synchronized (this) {
                if (engine == null) {
                    engine = createJavaScriptEngine("JavaScript");
                }
            }
        }
    }

    private ScriptEngine createJavaScriptEngine(String name) {
        ScriptEngineManager factory = new ScriptEngineManager();
        return factory.getEngineByName(name);
    }

    public void createNashornSandboxIfNotExist() {
        if (sandbox == null) {
            synchronized (this) {
                if (sandbox == null) {
                    sandbox = createSandbox();
                }
            }
        }
    }

    private NashornSandbox createSandbox() {
        NashornSandbox sandbox = NashornSandboxes.create();
        sandbox.setMaxCPUTime(3000); // 3s
        sandbox.setMaxMemory(1024 * 1024); // 1MB
        sandbox.allowNoBraces(false);
        sandbox.allowLoadFunctions(false);
        sandbox.setMaxPreparedStatements(30); // Cache LRU initial capacity
        sandbox.setExecutor(Executors.newSingleThreadExecutor());
        return sandbox;
    }

    public void compileAndCacheScript(String script) throws ScriptException {
        if (engine == null) {
            throw new RuntimeException("engine must not be null, please first create engine");
        }
        if (sandbox == null) {
            throw new RuntimeException("sandbox must not be null, please first create sandbox");
        }
        Compilable compilable = (Compilable) engine;
        // 预编译待执行脚本文件
        // CompiledScript compiledScript = compilable.compile(new FileReader("D:\\project-info\\do-parent\\do-api\\src\\main\\java\\com\\api\\utils\\test.js"));
        CompiledScript compiledScript = compilable.compile(script);
        // 执行脚本， 如果不在function内则执行了，否则不执行
        sandbox.eval(compiledScript);
    }

    public static void main(String[] args) throws Exception {
        JsUtils instance = JsUtils.getInstance();
        instance.createJavaScriptEngineIfNotExist();
        instance.createNashornSandboxIfNotExist();
        instance.compileAndCacheScript("var Ses = Java.type(\"com.api.service.impl.script.ScriptExecuteService\");\n" +
                "function test() {\n" +
                "    function test3() {\n" +
                "        print(ctx);\n" +
                "    }\n" +
                "\n" +
                "    test3();\n" +
                "    // print(ctx);  // RespDTO(resultCode=200, message=Ok, data=null)\n" +
                "    // print(typeof ctx); // object\n" +
                "    print('resultCode: ', ctx.resultCode);\n" +
                "    print(\"js 执行输出\");\n" +
                "\n" +
                "    print(\"js 执行Java Method输出--->\", Ses.testPrint(\"js\"));\n" +
                "    return \"hello javascript\";\n" +
                "}");
        // FileWriter fileWriter = new FileWriter("D:\\project-info\\do-parent\\do-api\\src\\main\\java\\com\\api\\utils\\output.txt");
        StringWriter output = new StringWriter();
        Object res = instance.executeScript("test", new Object[0], RespDTO.ok(), output);
        System.out.println(" ----- 输出结果打印 -----");
        System.out.println(res);
        System.out.println(" ----- 执行记录打印 -----");
        System.out.println(output);
    }

    public Object executeScript(String scriptMethodName, Object[] args, Object context, Writer execWriter) throws IOException {
        try {
            ScriptContext scriptContext = engine.getContext();
            scriptContext.setWriter(execWriter);
            scriptContext.setErrorWriter(execWriter);
            // 当前脚本引擎上下文
            scriptContext.setAttribute("ctx", context, ScriptContext.ENGINE_SCOPE);
            return ((Invocable) engine).invokeFunction(scriptMethodName, args);
        } catch (Exception e) {
            e.printStackTrace();
            execWriter.write(JSON.toJSONString(e));
        }
        return null;
    }
}
```

## 需求背景

	自定义代码检查，在编译阶段防止漏掉注解导致权限出现异常；所以在Controller层检查是否有权限注解；

## 实现步骤

```java
// 1 自定义注解：
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface CheckPermissionAnnoation {
}

//2 自定义权限注解
//此注解为必须需要权限的方法上贴此注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Inherited
public @interface PermissionAnnoation {

}




//此注解贴到需要检查权限注解的Controller上
// 例如
@CheckPermissionAnnoation
public class DemoController {
    @Resource
    private DemoService demoService;

    //该方法必须要注解，所以要贴此注解
    @PermissionAnnoation
    public  Wrapper<?> testMethod(){
        //service调用方法，自定义逻辑返回值

        return null;
    }
    //此方法也需要注解，但是不贴，代码检查会提示
    public  Wrapper<?> testMethod2(){
        //service调用方法，自定义逻辑返回值
        return null;
    }
}
```

## 自定义实现

```JAVA
@SupportedAnnotationTypes("com.test.annotation.CheckPermissionAnnoation")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class CheckPermissionAnnoationProcesser extends AbstractProcessor {


    private Messager messager;

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        this.messager = processingEnv.getMessager();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        //获取贴了CheckPermissionAnnoation注解的
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(CheckPermissionAnnoation.class);

        for (Element element : elements) {
            //获取方法
            for (ExecutableElement executableElement : ElementFilter.methodsIn(element.getEnclosedElements())) {
                messager.printMessage(Diagnostic.Kind.NOTE, "executableElement >>> " + executableElement);
                //获取是否贴了注解
                PermissionAnnoation permissionAnnoation = executableElement.getAnnotation(PermissionAnnoation.class);
                //没有贴就编译不通过  
                if (ObjectUtil.isNull(permissionAnnoation)) {
                    String formate = "this method must have annotation: PermissionAnnoation >>> %s";
                    String msg = element.getSimpleName().toString() + "." + executableElement.getSimpleName().toString();
                    messager.printMessage(Diagnostic.Kind.ERROR, String.format(formate, msg));
                    return false;
                }
            }
        }
        return true;
    }
}

```

## 添加文件

在resources/META-INF/services文件夹下新建文件javax.annotation.processing.Processor

文件内容为:
（自定时实现的AbstractProcesser的类的路径）

```
com.test.processor.CheckPermissionAnnoationProcesser
```

此时再编译即可检查哪些方法未贴需要贴的权限注解；

上面的代码在编译时就会报错：

```java
this method must have annotation: PermissionAnnoation >>>  DemoController.testMethod2;
```

## 完成

至于权限注解中需要怎么实现，使用AOP拦截即可；切面为贴了权限注解的地方；



*后续有时间的话写下权限拦截实现*
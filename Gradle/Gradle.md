![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic1.zhimg.com%2Fv2-0da0957e5b48dfdd8c975a53d21bcedb_400x224.jpg&refer=http%3A%2F%2Fpic1.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1659700023&t=cbb387c84fadfd1b4a70d6d6c6375fb2)



## 什么是Gradle

- Gradle是一个通用的构建工具;

- 核心模型基于任务（Task），意味着构建本质上是配置一组任务并将它们连接在一起;

  任务本身包括：

  - 动作——做某事的工作，比如复制文件或编译源代码
  - 输入——动作使用或操作的值、文件和目录
  - 输出——操作修改或生成的文件和目录

  ![](https://docs.gradle.org/current/userguide/img/task-dag-examples.png)

- Gradle有几个固定构建阶段

  - 初始化，为构建设置环境并确定哪些项目将参与其中。
  - 配置，为构建配置任务图，然后根据用户想要运行的任务确定需要运行哪些任务以及以何种顺序运行。
  - 执行，运行在配置阶段结束时选择的任务。

- Gradle 可以通过多种方式进行扩展
  - 自定义任务类型，构建完成一些现有任务无法完成的工作时，您可以简单地编写自己的任务类型，通常最好将自定义任务类型的源文件放在buildSrc目录或打包插件中
  - 自定义任务操作，通过任务的doFirst和doLast方法附加在任务之前或之后执行定义的构建漏记；
  - 项目和任务的额外属性，将自己的属性添加到项目或任务中；
  - 自定义约定，约定是一种简化构建的方法，编写自己的插件来提供约定，为构建的相关方面配置默认值。

## Groovy介绍

Groovy是JVM平台上的一种面向对象且同时支持静态动态的脚本语言，语法和Java区别不大，提供了一些语法糖，代码的表达能力更强。

```groovy
//1、可以用def关键字定义变量和方法，编译期做类型推断
//2、多变量同时创建
def (aa, bb) = [1, 2]
//3、范围创建
int[] range = 0..10;
//4、支持for in写法
for(int i in 1..5) {
    println(i);
}
//5、字符串支持单引号和双引号，双引号中可识别变量
def res='字符串'
println("打印结果是：${res}")
//6、列表创建
List<String> strings = ["g", "r", "o", "o", "v", "y"]
//7、map创建
Map<String, String> stringMap = ["name": "wang", "age": "99"]
//8、trait关键字声明一个可以有属性和默认实现的接口，Java8之后的接口也都能达到同样效果
trait Marks {
    void DisplayMarks() {
        println("Display Marks");
    }
}
class Student implements Marks {
    int StudentID
    int Marks1;

}

Student st = new Student();
st.StudentID = 1;
st.Marks1 = 10;
st.DisplayMarks();

//9、支持闭包，自己Call自己
def closure = { param -> println "Hello ${param}" };
closure.call("World");

//也可以省去单个参数，默认是it
def clos = {println "Hello ${it}"};
 clos.call("World");

10.times {num -> println num} 

//10、方法调用
class Student {
    String StudentID

    String getFun2(){
        return 'fun2';
    }

    void fun1(clo) {
        clo.call(StudentID);
    }
}

def student = new Student()
student.StudentID '99'
println student.fun2
println student.studentID
student.fun1 {println("${it}")}
```

## 了解生命周期

### 构建阶段

- 初始化，Gradle 支持单项目和多项目构建。在初始化阶段，Gradle 确定哪些项目将参与构建，并为每个项目创建一个Project实例。
- 配置，在此阶段配置项目对象。*执行作为构建一部分的所有*项目的构建脚本。
- 执行Gradle 确定在配置阶段创建和配置的要执行的任务子集。

### [Settings file](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:settings_file)

除了构建脚本文件，Gradle 还定义了一个设置文件。设置文件由 Gradle 通过命名约定确定。此文件的默认名称是`settings.gradle`。设置文件在初始化阶段执行，多项目构建必须在根目录下有settings.gradle。

```groovy
//settings.gradle
rootProject.name = 'basic'
println 'This is executed during the initialization phase.'

//build.gradle
println 'This is executed during the configuration phase.'

tasks.register('configured') {
    println 'This is also executed during the configuration phase, because :configured is used in the build.'
}

tasks.register('test') {
    doLast {
        println 'This is executed during the execution phase.'
    }
}

tasks.register('testBoth') {
	doFirst {
	  println 'This is executed first during the execution phase.'
	}
	doLast {
	  println 'This is executed last during the execution phase.'
	}
	println 'This is executed during the configuration phase as well, because :testBoth is used in the build.'
}
```

## 编写构建脚本

```groovy
//注册任务
tasks.register('count') {
    doLast {
        4.times { print "$it " }
    }
}

//任务依赖
tasks.register('hello') {
    doLast {
        println 'Hello world!'
    }
}
tasks.register('intro') {
    dependsOn tasks.hello
    doLast {
        println "I'm Gradle"
    }
}

//操作现有任务
4.times { counter ->
    tasks.register("task$counter") {
        doLast {
            println "I'm task number $counter"
        }
    }
}
tasks.named('task0') { dependsOn('task2', 'task3') }
```

## 使用Gradle插件

Gradle所有用的特性，比如编译java代码的能力，都是有插件添加的。

### 插件类型

Gradle 中有两种通用的插件类型，*二进制*插件和*脚本*插件。二进制插件可以通过实现[Plugin](https://docs.gradle.org/current/javadoc/org/gradle/api/Plugin.html)接口以编程方式编写，也可以使用 Gradle 的一种 DSL 语言以声明方式编写。脚本插件是额外的构建脚本，可以进一步配置构建

## 二进制插件

可以通过*插件 id*应用插件，这是插件的全局唯一标识符或名称。核心 Gradle 插件的特殊之处在于它们提供了短名称，例如`'java'`核心[JavaPlugin](https://docs.gradle.org/current/javadoc/org/gradle/api/plugins/JavaPlugin.html)。

```groovy
//应用核心插件
plugins {
    id 'java'
}
//应用社区插件
plugins {
    id 'com.jfrog.bintray' version '1.8.5'
}
```



## 开发Gradle任务

### [跳过任务](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:skipping_tasks)

#### [使用断言](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:using_a_predicate)

**使用onlyIf方法将断言添加到任务，只有断言执行的结果为true，任务才会执行**

```groovy
def hello = tasks.register('hello') {
    doLast {
        println 'hello world'
    }
}

hello.configure {
    onlyIf { !project.hasProperty('skipHello') }
}
```

```groovy
 ./gradlew -q hello -PskipHello
```

#### [使用 StopExecutionException](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:using_stopexecutionexception)

**如果某个动作抛出此异常，则跳过该动作的进一步执行以及该任务的任何后续动作的执行，构建继续执行下一个任务。**

```groovy
def compile = tasks.register('compile') {
    doLast {
        println 'We are doing the compile.'
    }
}

compile.configure {
    doFirst {
        // Here you would put arbitrary conditions in real life.
        if (true) {
            throw new StopExecutionException()
        }
    }
}
tasks.register('myTask') {
    dependsOn('compile')
    doLast {
        println 'I am not affected'
    }
}
```

```groovy
./gradlew -q myTask
```

#### [启用和禁用任务](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:enabling_and_disabling_tasks)

**每个任务都有一个`enabled`默认为 的标志`true`。将其设置为`false`阻止执行任何任务的操作。禁用的任务将被标记为 SKIPPED。**

 ```groovy
 def disableMe = tasks.register('disableMe') {
     doLast {
         println 'This should not be printed if the task is disabled.'
     }
 }
 
 disableMe.configure {
     enabled = false
 }
 ```

#### [任务规则](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:task_rules)

**有时您希望有一个任务，其行为取决于参数的大或无限数值范围。提供此类任务的一种非常好的和富有表现力的方式是任务规则**：

```groovy
//String 参数用作规则的描述
tasks.addRule("Pattern: ping<ID>") { String taskName ->

    if (taskName.startsWith("ping")) {
        task(taskName) {
            doLast {
                println "Pinging: " + (taskName - 'ping')
            }
        }
    }
}
```

#### [终结器任务](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:finalizer_tasks)

**当最终确定的任务计划运行时，终结器任务会自动添加到任务图中。即使最终确定的任务失败，也将执行终结器任务。**

```groovy
def taskX = tasks.register('taskX') {
    doLast {
        println 'taskX'
    }
}
def taskY = tasks.register('taskY') {
    doLast {
        println 'taskY'
    }
}

taskX.configure { finalizedBy taskY }
```

## 开发自定义 Gradle 任务类型

Gradle 支持两种类型的任务。一种这样的类型是简单任务，您可以在其中使用操作闭包定义任务。对于这种类型的任务，动作闭包决定了任务的行为。这种类型的任务非常适合在构建脚本中实现一次性任务。

另一种类型的任务是增强型任务，其中行为内置于任务中，并且任务提供了一些可用于配置行为的属性。

### [编写一个简单的任务类](https://docs.gradle.org/current/userguide/custom_tasks.html#sec:writing_a_simple_task_class)

**要实现自定义任务类，请扩展[DefaultTask](https://docs.gradle.org/current/dsl/org.gradle.api.DefaultTask.html)。**

```groovy
//build.gradle
abstract class GreetingTask extends DefaultTask {
    @TaskAction
    def fun1(){
        println 'hello from GreetingTask'
    }
}


// Create a task using the task type
tasks.register('hello', GreetingTask){
    doLast {
        println 'doLast from greetingTask'
    }
}
```

***一个可定制的 hello world 任务***

```groovy
////build.gradle
abstract class GreetingTask extends DefaultTask {
    @Input
    abstract Property<String> getGreeting()

    GreetingTask() {
        greeting.convention('hello from GreetingTask')
    }

    @TaskAction
    def greet() {
        println greeting.get()
    }
}

// Use the default greeting
tasks.register('hello', GreetingTask)

// Customize the greeting
tasks.register('greeting',GreetingTask) {
    greeting = 'greetings from GreetingTask'
}
```

> | 注解     | 预期的属性类型         | 描述             |
> | :------- | :--------------------- | :--------------- |
> | `@Input` | 任何`Serializable`类型 | 一个简单的输入值 |


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


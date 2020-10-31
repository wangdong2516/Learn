<center><h1> Gitlab CI

> 介绍

1. 如果要使用gitlab CI，你必须先有一个托管在gitlab上的代码仓库(必要条件)
2. 在项目根目录中，必须存在`.gitlab-ci.yml`的配置文件，该文件的作用是定义gitlab runner在提交代码的时候，应该执行的命令或者是脚本。
3. 在此文件中，你可以定义将要运行的脚本，执行的命令或者是需要缓存的依赖项
4. 将`gitlab-ci.yml`文件添加到存储库中的时候，Gitlab将会检测到该文件并且使用名为Gitlab Runner的脚本运行工具去运行这些命令，而Gitlab Runner本质上是脚本运行工具。

> 概念的介绍

1. CI:表示持续集成，意思是代码持续的和测试环境或者是生产环境进行合并
2. CD：表示持续交付或者是持续部署，指的是代码需要频繁的部署到测试环境或者是生产环境中。
3. 工作步骤:在gitlab-ci.yml文件中定义的工作步骤就是在进行CI的时候执行的步骤，具体的来讲就是在配置文件中使用`stages`关键字定义的步骤，这些步骤是顺序执行的。
4. 默认的工作步骤包括: `build`、`test`以及`depoly`三个，并且没有工作步骤将会被忽略。

> yml文件语法

1. `before_script`:在运行任何的工作步骤之前运行，全局定义的before_script的优先级小于在每个工作步骤中定义的before_script的优先级，也就是说，在工作步骤中定义的before_script将会覆盖全局定义的before_script的优先级。

2. 每个工作阶段必须有一个唯一的名称，并且有一些保留的关键字是不能用做作业的名称的

    ```yml
    image
    services
    stages
    types
    before_script
    after_script
    variables
    cache
    include
    ```

3. `script`：定义执行程序的过程中需要运行的脚本，每个作业必须有一个script才是有效的。必须是数组的形式。

4. `after_script`：在每个作业执行之后，运行的一组命令，和`before_script`不共用同一个shell，必须是数组的形式。

5. `allow_failure`：允许作业失败，失败的作业将不会影响最终的状态。默认是false，启用之后失败的作业将不会影响接下来的作业，但是会显示橙色的警告。

6. `artifacts`：作业执行成功之后，需要附加到作业的文件和目录名的列表。路径是相对于项目目录而言的，不能使用项目目录之外的文件或者是目录。可以使用通配符。

    + 子选项:
        + paths：将被添加到作业中的文件或者是目录的列表

    ```yml
    artifacts:
    	paths:
    		- cache/ 		
    ```

    + 子选项
        + exclude：不会被添加到作业中的文件或者是目录列表，用法和path相同。
        + 
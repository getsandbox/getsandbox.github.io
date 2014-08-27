---
layout: post
heading: Reducing Gradle memory usage during JavaExec.
description: Because of the way Gradle and Java fork child processes, if you use Gradle to start a JVM during development it can consumer a significant amount of memory, for no benefit. With a simple change you can reclaim your memory!
author: nick
---

### OutOfMemoryException

We develop [Sandbox](https://getsandbox.com) in a Vagrant virtualbox environment, this affords us many benefits like simple setup, faster developer onboarding etc, but running all the components of your application inside a virtual machine means it is going to have to live with less resources than if you were running it natively (generally). 

We recently started having slow downs and OutOfMemory exception errors during development and after a bit of investigation the root cause ended up being Gradle. We use Gradle for dependency management, some build tasks and most importantly we use it to start the JVM during development time using the `JavaExec` task.

The `JavaExec` task is a really convenient way of running your app with all its dependencies from the command line, in the same vein as a `mvn run/jettyRun`. The problem is that when Gradle runs a `JavaExec` task, it starts a Gradle JVM to resolve and compile the dependency graph and then forks a new child process to start your application, leaving the parent Gradle process running. In our case this parent Gradle process was taking up 200MB RAM, and depending on how many of the application components were running, there could be up to 4 Gradle processes running totaling **800MB of RAM wasted**. Our Vagrant VM only had 1500MB allocated, no wonder we were getting OOMs!

### The solution

We needed a solution where Gradle still maintained the dependency graph, but delegating the actual running of the JVM to the OS and not fork a child process itself. This is an inherent problem with Java, as the underlying Java mechanism of running processes `Runtime.exec(...)` doesn't support detached child processes.. anyway.

So to get around this we want Gradle to write out all the JARs (with absolute paths) that the application requires to run to a file, then get the OS to execute Java with that classpath. Here is quick and dirty version:

{% highlight groovy %}
task createdeps() {
    def file = new File("/tmp/runcomponent.sh")
    file.write("/usr/bin/java -cp " + sourceSets.main.runtimeClasspath.asPath + " com.test.YourMainClass");
}
{% endhighlight %}

We create a new task called 'createdeps' that will write a script to the `/tmp/` directory that can be executed to start a new JVM for our application. The script contains the full classpath as seen by Gradle, you should only have to replace the last parameter with your fully qualified Main class and it should work OTB! Because this Gradle task doesn't actually run anything, it just introspects the classpath, gradle should exit as soon as it has written the script, unlike a JavaExec task.

Tying it all together, a one-liner to write out the script and execute it:

{% highlight groovy %}
gradle createdeps; sh -c /tmp/runcomponent.sh
{% endhighlight %}

800MB RAM Saved!


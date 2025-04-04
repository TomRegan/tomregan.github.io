+++
date = '2012-08-06'
title = 'Using JVM Shutdown Hooks to Kill a Child Process'
+++

Today I spent some time writing a runner for a command-line Java socket 
application. I'd assumed that it would be easy enough to quit the process 
by hitting `^C`. I'm here to report that this is not the case.

Hitting `^C` shuts down the JVM, so if you've forked a separate
Java process it'll become orphaned.

To get past this you have to implement a shutdown hook, which will run
during the JVM shutdown sequence, in effect allowing your application
to handle the signal. Which is exactly what I wanted.

Here's a bit of code:

{% highlight java %}
class Runner {

    ...

    public int exec() {
        final String[] args = new String[] {
                "java",
                "-classpath", classpath,
                mainclass
        };
        
        final ProcessBuilder pd = new ProcessBuilder(args);
        pd.directory(new File(workingDirectory, "bin"));
        final Process p = pd.start();
        Runtime.getRuntime().addShutdownHook(new Thread() {

            public void run() {

                p.destroy();
            }
        });
        return p.waitFor();
    }
}
{% endhighlight %}

The Runtime API is [here](http://docs.oracle.com/javase/6/docs/api/java/lang/Runtime.html#addShutdownHook%28java.lang.Thread%29).

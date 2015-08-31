# Requirements #

In order to build Closure Stylesheets from source, you need to have the following tools installed:

  * [Git](http://git-scm.com/)
  * [Apache Ant](http://ant.apache.org/)

# Steps #

Once you have all of the above tools available from the command line, you can build Closure Stylesheets with the following commands:

```
git clone https://code.google.com/p/closure-stylesheets/
cd closure-stylesheets
ant
```

The compiled jar file will be available at `build/closure-stylesheets.jar`.

It is also a good idea to run all of the tests with the following command:

```
ant test
```

All tests should pass.

# Eclipse #

Closure Stylesheets has `.project` and `.classpath` files that contain the appropriate metadata to seamlessly import the source tree as a Java project in [Eclipse](http://www.eclipse.org/).

Note that the Eclipse project depends on the `build/genfiles/java` directory, which is not available unless `ant jar` has been run. You can either run this from the command line, or you can right-click on the project's `build.xml` file within Eclipse and choose **Run As** > **Ant Build** to generate the appropriate files.

You will likely need to refresh your project after running Ant so that Eclipse notices the newly created directory.
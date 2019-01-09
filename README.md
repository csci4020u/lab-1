# Lab 1: Development Environment

We will introduce:

- Git
- Java SDK
- Gradle
- JUnit

# Java

In this section, we will
develop a Java program that expects a file name as a command line argument.

- The program reads the file line by line.
- It prints each line by reversing the characters.

Consider a file

`input.txt`
```
hello
world
```

The application should be invoked as:

```
$ java App input.txt
olleh
dlrow
```

## Organizing your files

We wish to organize the source code and the compiled binary
as follows.

```
.
├── build
│   └── App.class
└── src
    └── main
        └── java
            └── App.java
```

So, you should first create the two directories:

```
$ mkdir -p ./build/ ./src/main/java/
```

## Programming

Now, we can write some Java code.

Complete the following Java class.  You can make changes if you wish.

`./src/main/java/App.java`

```java
import java.io.*;
import java.io.*;
import static java.lang.System.*

public class App {
    public static void main(String[] args) throws Exception {
        if(args.length == 0) {
            out.println("Usage: <filename>");
            exit(0);
        }

        String filename = args[0];

        BufferedReader in = new BufferedReader(new FileReader(filename));

        String line = in.readLine();
        while(line != null) {
            out.println(reverse(line));
            line = in.readLine();
        }
        in.close();
    }

    public static String reverse(String s) {
        /* You need to complete this */
    }
}
```

## Compilation and Execution

Let's create a simple text file.  *Note*: This file should contain only **two**
lines.

`./input.txt`
```
hello
world
```

Let's compile the Java program:

```
$ javac -d ./build ./src/main/java/App.java
```

If nothing goes wrong, we can try it out:

```
$ java -cp ./build App input.txt
```

You should expect:

```
olleh
dlrow
```

# Gradle

It's rather inconvenient to develop Java programs.  Any time we modify the
source code, we must remember to recompile the class files before running the
application.  As more classes are created, it can be a hassle to keep track of
which classes must be recompiled.

Gradle helps us to manage the compilation process, and much more as we will
discover later in the course.

## Compilation with Gradle

Let's author a `./build.gradle` file that looks like this:

`./build.gradle`

```
apply plugin: 'java'
```

So, the directory structure looks like:

```
.
├── build.gradle
├── input.txt
└── src
    └── main
        └── java
            └── App.java
```

Now, compilation is _easy_:

```
$ gradle build
```

This generates the Java class files (and more):

```
.
├── build
│   ├── classes
│   │   └── java
│   │       └── main
│   │           └── App.class
│   ├── libs
│   │   └── lab-1.jar
│   └── tmp
│       ├── compileJava
│       └── jar
│           └── MANIFEST.MF
├── build.gradle
├── input.txt
└── src
    └── main
        └── java
            └── App.java
```

Note that:

- Gradle compiled the class file, **and** generated
a Java archive file (.jar file).

We can run the application as before:

```
$ java -cp ./build App input.txt
```

We can also use the _library_:

```
$ java -cp ./build/libs/lab-1.jar App input.txt
```

Try 

```
gradle clean
```

It removes all compiled outputs, so `build` is gone.  Don't worry, you can
simply do:

```
gradle build
```

to regenerate everything.

## Execution with Gradle

We can also use gradle to execute the application.  This requires us
to tell Gradle a few more information.  We need to modify `build.gradle` as
follows:

`build.gradle`

```
plugins {
  id 'application'
}

application {
  mainClassName = 'App'
}

run {
  args = ['input.txt']
}

apply plugin: 'java'
```

Now, we can ask gradle to do many things:

```
$ gradle build
$ gradle run
$ gradle clean
```

Gradle is intelligent to do minimal amount of recompilation.  So, if you
try to build multiple times, it will only trigger recompilation once.

Try this:
```
$ gradle build
$ gradle build
$ gradle build
```

# Unit Testing with Gradle

The application consists of one class and two methods:

```
class App {
    public static void main(String[] args) ...
    public static String reverse(String s) ....
}
```

We are going to write some unit tests.

1. Make sure that reverse handles null string properly.
2. Make sure that reverse handles empty string properly.
3. Make sure that main handles non-existing files.
4. ...


JUnit is a great library to do unit tests.  It exists as a library, and is
available on the _maven repository_.  We will use JUnit 4.

Gradle is great because:

1. It helps us download the library if necessary.
2. It runs all the unit tests.  We have to write them though.
3. It compiles the test results into a report.

## Update `build.gradle`

`build.gradle`

```
plugins {
  id 'application'
}

repositories {
  mavenCentral()
}

dependencies {
  compile 'junit:junit:4.12'
  testCompile 'junit:junit:4.12'
}

application {
  mainClassName = 'App'
}

run {
  args = ['input.txt']
}

apply plugin: 'java'
```

Even without any unit tests, we can still ask gradle to perform unit tests:

```
$ gradle test
```

## Writing Unit Tests

We need to put the tests at the specific directory: `./src/tests/java/...`.

Let's write a trivial test case.

`./src/tests/java/TestApp.java`

```
import org.junit.*;
import static org.junit.Assert.*;

public class TestApp {
    @Test
    public void Test0() {
        assertTrue("Just checking...", true);
    }
}
```

So, the directory structure looks like:

```
./src
├── main
│   └── java
│       └── App.java
└── test
    └── java
        └── TestApp.java
```

Now, we can ask gradle to perform the tests (just one for now).

```
$ gradle test
```

Everything should pass. The report is available:

```
build
├── reports
│   └── tests
│       └── test
│           ├── classes
│           │   └── TestApp.html
│           ├── css
│           │   ├── base-style.css
│           │   └── style.css
│           ├── index.html
│           ├── js
│           │   └── report.js
│           └── packages
│               └── default-package.html
├── test-results
│   └── test
│       ├── binary
│       │   ├── output.bin
│       │   ├── output.bin.idx
│       │   └── results.bin
│       └── TEST-TestApp.xml
```

Just open the HTML file:

```
$ open build/report/tests/test/index.html
```

## Writing More Unit Tests

`TestApp.java`

```
import org.junit.*;
import static org.junit.Assert.*;

public class TestApp {
    @Test
    public void Test0() {
        assertTrue("Just checking...", true);
    }

    @Test
    public void TestReverse() {
        assertEquals(App.reverse(""), "");
        assertEquals(App.reverse(null), null);
        assertEquals(App.reverse("abc"), "cba");
    }

    @Test
    public void TestMain() {
        try {
            App.main(new String[] {"this-does-not-exist"});
            fail("Should throw exception");
        } catch(Exception e) {
            assertTrue(true);
        }
    }

}
```

# Conclusion

In this lab, you should be able to:

1. Setup the directory structure and `build.gradle` to develop
Java applications using gradle.

2. Understand JUnit and Gradle

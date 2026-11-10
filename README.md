# linkja-salt-engine

## Building
linkja-salt-engine needs Java 25 (e.g. [Eclipse Temurin](https://adoptium.net/)) and [Maven](https://maven.apache.org/).

The tests call the linkja C library. The `lib` folder holds a Windows build and an Intel-only macOS build. On any other machine, compile the library from the `develop` branch of [linkja-crypto](https://github.com/MRAIA-ORG/linkja-crypto). The public build is enough, because linkja-salt-engine uses only functions that it holds (token generation and the library signature). It needs OpenSSL 3, CMake and cmocka (e.g. `brew install openssl@3 cmake cmocka`). From a linkja-crypto clone beside this one, with `JAVA_HOME` set to Java 25:

```
cmake . -DINCLUDE_SECRETS=OFF -DCMAKE_BUILD_TYPE=Release
make linkjacrypto
```

On macOS, add `-DOPENSSL_ROOT_DIR="$(brew --prefix openssl@3)"` to the `cmake` command.

Then, from this folder:

`mvn clean package -Dnative.lib.dir=../linkja-crypto/target`

This compiles the code, runs all unit tests, and puts the JARs in the `target` folder, e.g. `target/SaltEngine-0.5-jar-with-dependencies.jar`, which holds all dependency JARs. If you do not give `native.lib.dir`, the tests look for the library in the `lib` folder.

CI runs the same steps on each pull request. See `.github/workflows/ci.yml`.

## Program Use
You can run the executable JAR file using the standard Java command:
`java -jar SaltEngine-1.0-jar-with-dependencies.jar `

The program has two modes that can be invoked - the primary one (enabled with `--generateProject`) will create a set of salt files for multiple sites in a project.
The second mode allows you to add one or more sites to an existing project (enabled with `--addSites`).

For hashing, linkja-salt-engine uses a special C library (.dll/.dylib/.so, depending on your operating system).  You will need to tell Java where to find this library when you try to run the program.  Otherwise, you will get an error:

```
Exception in thread "main" java.lang.UnsatisfiedLinkError: no linkjacrypto in java.library.path:
```

A redistributable version of the library is provided in this repository under the `lib` folder.  The library may be placed in any directory found by the Java library path.  If you would like to specify the library, you can include the `-Djava.library.path=` option when running the program.
This can be the same directory as the linkja-salt-engine JAR file (e.g., `-Djava.library.path=.`).

If you specify `--version`, the program will display the application version.  

Note that where files are used for input, they can be specified as a relative or absolute path.


### Generate Project
Usage: `java -jar SaltEngine.jar --generateProject`

The program is expecting a minimum of two parameters:

```
 -sf,--siteFile <arg>              The path to a file containing the site definitions
 -pn,--projectName <arg>           The name of the project to create
```

### Add Sites to Existing Project
Usage: `java -jar SaltEngine.jar --addSites`

The program is expecting a minimum of three parameters:

```
 -sf,--siteFile <arg>              The path to a file containing the site definitions
 -salt,--saltFile <arg>            The path to your encrypted salt file for the existing project
```


**Examples:**

Display the version information

```
java -Djava.library.path=. -jar SaltEngine-1.0-jar-with-dependencies.jar --version
```

Create salt files for a project named 'Test Project', for all sites specified in a
file named `sites.csv`.  The CSV file is located in a subdirectory (`data`) relative to 
where the JAR is located.

```
java -Djava.library.path=. -jar SaltEngine-1.0-jar-with-dependencies.jar --generateProject
  --siteFile data/sites.csv --projectName "Test Project"
```

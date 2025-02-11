---
layout: default
title: Compiling Babelfish from source
nav_order: 2
---

# Compiling Babelfish from source

If you want to compile Babelfish on your own, you need to follow a couple of steps to
produce a working binary. In this section, we will describe how this works on
Linux (and thus UNIX-style systems in general). 


## Getting the source code

The first thing to do is to actually download the source code you want to build.
Babelfish is separated into two repositories, the first of them contains 
the PostgreSQL database engine, with some changes that enables the 
protocols, language parsers, and more features to be hooked into PostgreSQL that are required by Babelfish to work. 
The second one contains extensions to support the T-SQL protocol, the T-SQL language, the TDS Protocol, etc.

The Babelfish for PostgreSQL engine source code can be downloaded 
from [here](https://github.com/babelfish-for-postgresql/postgresql_modified_for_babelfish).

The Babelfish extensions source code can be downloaded 
from [here](https://github.com/babelfish-for-postgresql/babelfish_extensions).

If you have git installed, you can clone the repos with the following command:

``` sh
git clone https://github.com/babelfish-for-postgresql/postgresql_modified_for_babelfish.git
git clone https://github.com/babelfish-for-postgresql/babelfish_extensions.git
```


## Preparing your system

To compile Babelfish, you have to make sure that a variety of software components
are available on your system. These packages should be part of every modern
Linux distribution, under similar - but not identical - names. 

Also keep in mind that you will need a user with root privileges, so that you are able to run commands with `sudo`

If you want to run more than just the bare version of Babelfish, we recommend
installing the following additional packages on top of the hard requirements listed
above.

- [Flex 2.6.4](https://github.com/westes/flex)
- [Libxml2](http://xmlsoft.org/) development libraries
- [Open SSL](https://www.openssl.org/) development libraries
- [Readline](https://tiswww.cwru.edu/php/chet/readline/rltop.html) development libraries
- [Zlib](https://zlib.net/)
- [OSSP uuid](http://www.ossp.org/pkg/lib/uuid/) development libraries
- [pkg-config](https://linux.die.net/man/1/pkg-config)
- [ICU](https://icu.unicode.org/) development libraries
- [Bison 3.0.5 or higher](https://www.gnu.org/software/bison/)

If you happen to use Debian or Ubuntu, you might want to use the following
commands to install dependencies:

``` sh
sudo apt install -y build-essential flex libxml2-dev bison libreadline-dev zlib1g-dev
sudo apt install -y uuid-dev pkg-config libossp-uuid-dev libssl-dev icu-devtools
```

## Compiling the code 

Before compiling the Babelfish for PostgreSQL source code, we need to configure the
build. To do that we need to run the `configure` script in the directory where you
have downloaded the Babelfish for PostgreSQL engine sources:

``` sh
./configure CFLAGS="-ggdb" \
  --enable-debug \
  --with-libxml \
  --with-uuid=ossp \
  --with-icu \
  --with-extra-version=" Babelfish for PostgreSQL"
``` 

The above would configure the installation path under `/usr/local/pgsql`; if
you want to change the installation directory, you can use the `prefix` flag.
Therefore, if you want to change the path to `/usr/local/pgsql-13.4`, you can
run the configure script as follows.

 ``` sh
 ./configure CFLAGS="-ggdb" \
  --prefix=/usr/local/pgsql-13.4 \
  --enable-debug \
  --with-libxml \
  --with-uuid=ossp \
  --with-icu \
  --with-extra-version=" Babelfish for PostgreSQL"
 ```

#### Building Babelfish for PostgreSQL engine

Now that we have configured the source tree, is important to configure the installation folder. 

``` sh
INSTALLATION_PATH=<the path you specified as prefix>
mkdir "$INSTALLATION_PATH"
```

To avoid installation errors, you should own the installation directory. You can can change the 
 ownership of the installation path with the following command:

``` sh
sudo chown -R <your user>:<your group> "$INSTALLATION_PATH"
```

For example, if your installation path is `/usr/local/pgsql-13.4` and your user is `johndoe`, 
 the command should be as follows:

``` sh
sudo chown -R johndoe:johndoe /usr/local/pgsql-13.4
```

Now we can build Babelfish with the following commands:

``` sh
make            # Compiles the Babefish for PostgreSQL engine
cd contrib 

make            # Compiles the PostgreSQL default extensions
cd ..

make install    # Installs the Babelfish for PostgreSQL engine
cd contrib

make install    # Installs the PostgreSQL default extensions
```

#### Building Babelfish Extensions

In order to build the extensions we would need to install some additional tools: 

##### Additional requied tools
- [Antlr 4.9.2 Runtime](https://www.antlr.org/)
- [Open Java 8](https://openjdk.java.net/)
- Unzip
- [pkgconf](http://pkgconf.org/)
- libutfcpp development libraries
- [CMake](https://cmake.org/)

You can install most of these tools by running the command:

``` sh
sudo apt install -y openjdk-8-jre unzip libutfcpp-dev cmake curl
```

##### Installing Antlr4 runtime

> For Antlr4 4.9.2 Runtime, there are no available binaries for C++ in Ubuntu Focal, so it's necessary to compile it from source. Versions below 4.9 have not been fully tested yet. 

To install the Antlr4 runtime, we need to have the Antlr4 .jar. Babelfish extensions source code includes 
this .jar in the path `/contrib/babelfishpg_tsql/antlr/thirdparty/antlr`.

Keeping this in mind, we can install Antlr4 runtime by running:

``` sh
# Dowloads the compressed Antlr4 Runtime sources on /opt/antlr4-cpp-runtime-4.9.2-source.zip 
sudo curl https://www.antlr.org/download/antlr4-cpp-runtime-4.9.2-source.zip \
  --output /opt/antlr4-cpp-runtime-4.9.2-source.zip 

# Uncompress the source into /opt/antlr4
sudo unzip -d /opt/antlr4 /opt/antlr4-cpp-runtime-4.9.2-source.zip

sudo mkdir /opt/antlr4/build 
cd /opt/antlr4/build

EXTENSIONS_SOURCE_CODE_PATH="<the patch in which you downloaded the Babelfish extensions source code>"

# Generates the make files for the build
sudo cmake .. -DANTLR_JAR_LOCATION="$EXTENSIONS_SOURCE_CODE_PATH/contrib/babelfishpg_tsql/antlr/thirdparty/antlr/antlr-4.9.2-complete.jar" \
         -DCMAKE_INSTALL_PREFIX=/usr/local -DWITH_DEMO=True
# Compiles and install
sudo make
sudo make install
```

Now that we have the antlr4 runtime installed, we need to copy the
`libantlr4-runtime.so.4.9.2` library into the installed Babelfish for PostgreSQL
engine libs folder. We can do that by running the following command:

``` sh
sudo cp /usr/local/lib/libantlr4-runtime.so.4.9.2 "$INSTALLATION_PATH/lib"
```


# Build and install the extensions

Now that we have all of the tools installed to build the Babelfish extension, we
need to configure some environment variables: 

- `PG_CONFIG`: should point to the location of the pg_config file in the
  Babelfish for PostgreSQL engine installation, in our case: `$INSTALLATION_PATH/bin/pg_config`.

- `PG_SRC`: should point to the location of the Babelfish for PostgreSQL engine
  source folder.

- `cmake`: should contain the path of the cmake binary

Supposing that you have installed the Babelfish for PostgreSQL engine in
`/usr/local/pgsql-13.4/`, you have downloaded the Babelfish for PostgreSQL engine
source code in `~/postgresql_modified_for_babelfish`, and cmake is installed in
`/usr/local/bin/cmake`, the environment variables set up would be like this:

``` sh
export PG_CONFIG=/usr/local/pgsql-13.4/bin/pg_config
export PG_SRC=$HOME/postgresql_modified_for_babelfish
export cmake=/usr/local/bin/cmake
```

Now we are all set to build the extensions. To do so, we need to go to the
contrib folder in the Babelfish extension source code, and then build the
extensions one by one. We do it with the following script:

``` sh
# Install babelfishpg_money extension
cd contrib/babelfishpg_money
make
make install

# Install babelfishpg_common extension
cd ../babelfishpg_common
make 
make install

# Install babelfishpg_tds extension
cd ../babelfishpg_tds
make 
make install

# Installs the babelfishpg_tsql extension
cd ../babelfishpg_tsql
make 
make install
```

Once all extensions have been compiled you can start PostgreSQL manually.

## Further installations steps

Before starting Babelfish we need to do some changes in the installation folder. 
 This is because, PostgreSQL wont start if its owner has root access. Also, we need to create 
 a directory for PostgreSQL to store its data. 

Let's first, create the data directory, in this example we will use `/usr/local/pgsql/data` as the data 
 folder. 

``` sh
sudo mkdir -p /usr/local/pgsql/data
```

Now, let's create a postgres user

``` sh
sudo adduser postgres
```

With the created user, we can change the ownership the of the Babelfish binaries, and the data directory.

``` sh
sudo chown -R postgres:postgres $INSTALLATION_PATH
sudo chown -R postgres:postgres /usr/local/pgsql/data
```

Now we can use the created postgres user to initialize the database directory.

``` sh
sudo su postgres
$INSTALLATION_PATH/bin/initdb -D /usr/local/pgsql/data
```

Once the data directory has been initialized, we need to change the postgresql.conf file, by uncommenting and setting the following properties:

``` conf
listen_addresses = '*'
shared_preload_libraries = 'babelfishpg_tds'
```

Now you can start Babelfish by running the command:
``` sh
$INSTALLATION_PATH/bin/pg_ctl -D /usr/local/pgsql/data start
```

To finish the Babelfish setup, continue [here](/docs/installation/single-multiple)
with the initialization.

# MAC
```sh
brew install tomcat-native
==> Installing dependencies for tomcat-native: openssl, apr
==> Installing tomcat-native dependency: openssl
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/op
######################################################################## 100.0%
==> Pouring openssl-1.0.2k.sierra.bottle.tar.gz
==> Using the sandbox
==> Caveats
A CA file has been bootstrapped using certificates from the SystemRoots
keychain. To add additional certificates (e.g. the certificates added in
the System keychain), place .pem files in
  /usr/local/etc/openssl/certs

and run
  /usr/local/opt/openssl/bin/c_rehash

This formula is keg-only, which means it was not symlinked into /usr/local.

Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries

If you need to have this software first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile

For compilers to find this software you may need to set:
    LDFLAGS:  -L/usr/local/opt/openssl/lib
    CPPFLAGS: -I/usr/local/opt/openssl/include

==> Summary
🍺  /usr/local/Cellar/openssl/1.0.2k: 1,696 files, 12MB
==> Installing tomcat-native dependency: apr
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ap
######################################################################## 100.0%
==> Pouring apr-1.5.2_3.sierra.bottle.tar.gz
==> Caveats
This formula is keg-only, which means it was not symlinked into /usr/local.

Apple's CLT package contains apr

If you need to have this software first in your PATH run:
  echo 'export PATH="/usr/local/opt/apr/bin:$PATH"' >> ~/.bash_profile

==> Summary
🍺  /usr/local/Cellar/apr/1.5.2_3: 56 files, 1.2MB
==> Installing tomcat-native 
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/to
######################################################################## 100.0%
==> Pouring tomcat-native-1.2.10.sierra.bottle.tar.gz
==> Caveats
In order for tomcat's APR lifecycle listener to find this library, you'll
need to add it to java.library.path. This can be done by adding this line
to $CATALINA_HOME/bin/setenv.sh

  CATALINA_OPTS="$CATALINA_OPTS -Djava.library.path=/usr/local/opt/tomcat-native/lib"

If $CATALINA_HOME/bin/setenv.sh doesn't exist, create it and make it executable.
==> Summary
🍺  /usr/local/Cellar/tomcat-native/1.2.10: 11 files, 376.2KB
```
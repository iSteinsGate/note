### Java环境配置

1. 配置JAVA_HOME

   ```
   C:\Program Files\Java\jdk1.8.0_91
   ```

2. 配置CLASSPATH

   ```
   .;%JAVA_HOME%\bin;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar
   ```

3. 配置PATH

   ```
   %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
   ```
   
### linux 配置

   输入 vim /etc/profile 加入
   ```
   # java dev
      export JAVA_HOME=/usr/local/java/jdk1.8.0_181
      export JRE_HOME=${JAVA_HOME}/jre
      export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
      export PATH=${JAVA_HOME}/bin:$PATH
  ```

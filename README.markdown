Template for Hudson Jobs for PHP Projects
=========================================

[Hudson](http://hudson-ci.org/) is an extensible [continuous integration](http://martinfowler.com/articles/continuousIntegration.html) server that starts to see widespread adoption in the PHP community.
The goal of the `php-hudson-template` project is to provide a standard template for Hudson jobs for PHP projects.

Required Hudson Plugins
-----------------------

You need to install the following plugins for Hudson:

* [Checkstyle](http://wiki.hudson-ci.org/display/HUDSON/Checkstyle+Plugin) (for processing `PHP_CodeSniffer` logfiles in Checkstyle format)
* [Clover](http://wiki.hudson-ci.org/display/HUDSON/Clover+Plugin) (for processing PHPUnit code coverage xml output)
* [DRY](http://wiki.hudson-ci.org/display/HUDSON/DRY+Plugin) (for processing `phpcpd` logfiles in PMD-CPD format)
* [HTML Publisher](http://wiki.hudson-ci.org/display/HUDSON/HTML+Publisher+Plugin) (for publishing the PHPUnit code coverage report, for instance)
* [JDepend](http://wiki.hudson-ci.org/display/HUDSON/JDepend+Plugin) (for processing `PHP_Depend` logfiles in JDepend format)
* [Plot](http://wiki.hudson-ci.org/display/HUDSON/Plot+Plugin) (for processing `phploc` CSV output)
* [PMD](http://wiki.hudson-ci.org/display/HUDSON/PMD+Plugin) (for processing `phpmd` logfiles in PMD format)
* [Violations](http://wiki.hudson-ci.org/display/HUDSON/Violations) (for processing various logfiles)
* [xUnit](http://wiki.hudson-ci.org/display/HUDSON/xUnit+Plugin) (for processing PHPUnit logfiles in JUnit format)

You can install these plugins using the webfrontend http://SERVER/pluginManager/available or using the hudson-cli

Download: http://SERVER/jnlpJars/hudson-cli.jar and execute

    java -jar hudson-cli.jar -s http://SERVER install-plugin checkstyle
    java -jar hudson-cli.jar -s http://SERVER install-plugin clover
    java -jar hudson-cli.jar -s http://SERVER install-plugin dry
    java -jar hudson-cli.jar -s http://SERVER install-plugin htmlpublisher
    java -jar hudson-cli.jar -s http://SERVER install-plugin jdepend
    java -jar hudson-cli.jar -s http://SERVER install-plugin plot
    java -jar hudson-cli.jar -s http://SERVER install-plugin pmd
    java -jar hudson-cli.jar -s http://SERVER install-plugin violations
    java -jar hudson-cli.jar -s http://SERVER install-plugin xunit
    java -jar hudson-cli.jar -s http://SERVER safe-restart


Required PHP Tools
------------------

    pear channel-discover pear.pdepend.org 
    pear channel-discover pear.phpmd.org 
    pear channel-discover pear.phpunit.de
    pear channel-discover components.ez.no
    pear channel-discover pear.symfony-project.com

    pear install pdepend/PHP_Depend-beta
    pear install phpmd/PHP_PMD-alpha
    pear install phpunit/phpcpd
    pear install phpunit/phploc
    pear install PHPDocumentor
    pear install PHP_CodeSniffer
    pear install --alldeps phpunit/PHP_CodeBrowser
    pear install --alldeps phpunit/PHPUnit

Build Automation
----------------

For example, refer to the `build.xml` script of the [Object_Freezer](http://github.com/sebastianbergmann/php-object-freezer) project:

    <project name="php-object-freezer" default="build" basedir=".">
     <target name="clean">
      <!-- Clean up -->
      <delete dir="${basedir}/build"/>

      <!-- Create build directories -->
      <mkdir dir="${basedir}/build/api"/>
      <mkdir dir="${basedir}/build/code-browser"/>
      <mkdir dir="${basedir}/build/coverage"/>
      <mkdir dir="${basedir}/build/logs"/>
     </target>

     <!-- Run unit tests and generate junit.xml and clover.xml -->
     <target name="phpunit">
      <exec executable="phpunit" failonerror="true"/>
     </target>

     <!-- Run pdepend, phpmd, phpcpd, phpcs, phpdoc and phploc in parallel -->
     <target name="parallelTasks">
      <parallel>
       <antcall target="pdepend"/>
       <antcall target="phpmd"/>
       <antcall target="phpcpd"/>
       <antcall target="phpcs"/>
       <antcall target="phpdoc"/>
       <antcall target="phploc"/>
      </parallel>
     </target>

     <!-- Generate jdepend.xml and software metrics charts -->
     <target name="pdepend">
      <exec executable="pdepend">
       <arg line="--jdepend-xml=${basedir}/build/logs/jdepend.xml Object" />
      </exec>
     </target>

     <!-- Generate pmd.xml -->
     <target name="phpmd">
      <exec executable="phpmd">
       <arg line="Object xml codesize,unusedcode
                  --reportfile ${basedir}/build/logs/pmd.xml" />
      </exec>
     </target>

     <!-- Generate pmd-cpd.xml -->
     <target name="phpcpd">
      <exec executable="phpcpd">
       <arg line="--log-pmd ${basedir}/build/logs/pmd-cpd.xml Object" />
      </exec>
     </target>

     <!-- Generate phploc.csv -->
     <target name="phploc">
      <exec executable="phploc">
       <arg line="--log-csv ${basedir}/build/logs/phploc.csv Object" />
      </exec>
     </target>

     <!-- Generate checkstyle.xml -->
     <target name="phpcs">
      <exec executable="phpcs" output="/dev/null">
       <arg line="--report=checkstyle
                  --report-file=${basedir}/build/logs/checkstyle.xml
                  --standard=Sebastian
                  Object" />
      </exec>
     </target>

     <!-- Generate API documentation -->
     <target name="phpdoc">
      <exec executable="phpdoc">
       <arg line="-d Object -t ${basedir}/build/api" />
      </exec>
     </target>

     <target name="phpcb">
      <exec executable="phpcb">
       <arg line="--log    ${basedir}/build/logs
                  --source ${basedir}/Object
                  --output ${basedir}/build/code-browser" />
      </exec>
     </target>

     <target name="build" depends="clean,parallelTasks,phpunit,phpcb"/>
    </project>

The `build.xml` script above assumes that an XML configuration file for PHPUnit is used to configure the following logging targets:

    <logging>
     <log type="coverage-html" target="build/coverage" title="Object_Freezer"
          charset="UTF-8" yui="true" highlight="true"
          lowUpperBound="35" highLowerBound="70"/>
     <log type="coverage-clover" target="build/logs/clover.xml"/>
     <log type="junit" target="build/logs/junit.xml" logIncompleteSkipped="false"/>
    </logging>

Executing the `build.xml` script above will produce the following `build` directory:

    build
    |-- api ...
    |-- code-browser ...
    |-- coverage ...
    `-- logs
        |-- checkstyle.xml
        |-- clover.xml
        |-- jdepend.xml
        |-- junit.xml
        |-- phploc.csv
        |-- pmd-cpd.xml
        `-- pmd.xml

These build artifacts will be processed by Hudson.

Using the Job Template
----------------------

Check out `php-hudson-template` from Git:

    cd $HUDSON_HOME/jobs
    git clone git://github.com/sebastianbergmann/php-hudson-template.git php-template

* Restart Hudson.
* Click on "New Job".
* Enter a "Job name".
* Select "Copy existing job" and enter "php-template" into the "Copy from" field.
* Click "OK".
* Fill in your "Source Code Management" information.
* Configure a "Build Trigger", for instance "Poll SCM".
* Configure an Ant-based build.
* Click "Save".

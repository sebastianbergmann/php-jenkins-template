Template for Hudson Jobs for PHP Projects
=========================================

[Hudson](http://hudson-ci.org/) is an extensible [continuous integration](http://martinfowler.com/articles/continuousIntegration.html) server that starts to see widespread adoption in the PHP community.
The goal of the `php-hudson-template` project is to provide a standard template ([Template Project Plugin for Hudson](http://wiki.hudson-ci.org//display/HUDSON/Template+Project+Plugin)) for Hudson jobs for PHP projects.

Required Hudson Plugins
-----------------------

You need to install the following plugins for Hudson:

* [Checkstyle](http://wiki.hudson-ci.org/display/HUDSON/Checkstyle+Plugin) (for processing `PHP_CodeSniffer` logfiles in Checkstyle format)
* [DRY](http://wiki.hudson-ci.org/display/HUDSON/DRY+Plugin) (for processing `phpcpd` logfiles in PMD-CPD format)
* [HTML Publisher](http://wiki.hudson-ci.org/display/HUDSON/HTML+Publisher+Plugin) (for publishing the PHPUnit code coverage report, for instance)
* [JDepend](http://wiki.hudson-ci.org/display/HUDSON/JDepend+Plugin) (for processing `PHP_Depend` logfiles in JDepend format)
* [PMD](http://wiki.hudson-ci.org/display/HUDSON/PMD+Plugin) (for processing `phpmd` logfiles in PMD format)
* [Template Project](http://wiki.hudson-ci.org/display/HUDSON/Template+Project+Plugin) (for using `php-hudson-template` as a template for Hudson jobs)
* [Violations](http://wiki.hudson-ci.org/display/HUDSON/Violations) (for processing various logfiles)
* [xUnit](http://wiki.hudson-ci.org/display/HUDSON/xUnit+Plugin) (for processing PHPUnit logfiles in JUnit format)

Required PHP Tools
------------------

    pear channel-discover pear.pdepend.org 
    pear channel-discover pear.phpmd.org 
    pear channel-discover pear.phpunit.de
    pear channel-discover components.ez.no

    pear install pdepend/PHP_Depend-beta
    pear install --alldeps phpmd/PHP_PMD-alpha
    pear install phpunit/phpcpd
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
      <delete dir="build"/>

      <!-- Create build directories -->
      <mkdir dir="${basedir}/build/api"/>
      <mkdir dir="${basedir}/build/code-browser"/>
      <mkdir dir="${basedir}/build/coverage"/>
      <mkdir dir="${basedir}/build/logs"/>
      <mkdir dir="${basedir}/build/pdepend"/>
     </target>

     <!-- Run unit tests and generate junit.xml and clover.xml -->
     <target name="phpunit">
      <exec executable="phpunit" failonerror="true"/>
     </target>

     <!-- Run pdepend, phpmd, phpcpd, and phpcs in parallel -->
     <target name="parallelTasks">
      <parallel>
       <antcall target="pdepend"/>
       <antcall target="phpmd"/>
       <antcall target="phpcpd"/>
       <antcall target="phpcs"/>
       <antcall target="phpdoc"/>
      </parallel>
     </target>

     <!-- Generate jdepend.xml and software metrics charts -->
     <target name="pdepend">
      <exec executable="pdepend">
       <arg line="--jdepend-xml=${basedir}/build/logs/jdepend.xml
                  --jdepend-chart=${basedir}/build/pdepend/08-dependencies.svg
                  --overview-pyramid=${basedir}/build/pdepend/09-overview-pyramid.svg
                  Object" />
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
       <arg line="-d Object -t build/api" />
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

Executing the `build.xml` script above will produce the following `build` directory:

    build
    |-- api ...
    |-- code-browser ...
    |-- coverage ...
    |-- logs
    |   |-- checkstyle.xml
    |   |-- clover.xml
    |   |-- jdepend.xml
    |   |-- junit.xml
    |   |-- pmd-cpd.xml
    |   `-- pmd.xml
    `-- pdepend
        |-- 08-dependencies.svg
        `-- 09-overview-pyramid.svg

These build artifacts will be processed by Hudson.

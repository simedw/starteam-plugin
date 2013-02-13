Jenkins StarTeam plugin
=======================

Install
-------
This plugin requires a little extra attention to build.  As far as I can tell,
Borland does not publish their SDK in a maven repo.  In order to build, you
need to download the SDK from
http://www.borland.com/products/downloads/download_starteam.html.
In order to get jsafe.jar, one of the requirements, you must download the unix clients.

Then:
```bash
tar xzvf starteam-eval-unix-clients.tar.gz.gz
tar xzvf st-cpc-2009-en-universal.tar.gz
cd starteam-en-11.0.0-java/lib
mvn install:install-file -DgroupId=com.borland -DartifactId=starteam104 -Dpackaging=jar -Dversion=10 -Dfile=starteam104.jar
mvn install:install-file -DgroupId=com.borland -DartifactId=starteam104-resources -Dpackaging=jar -Dversion=10 -Dfile=starteam104-resources.jar
mvn install:install-file -DgroupId=com.smartsockets -DartifactId=ss -Dpackaging=jar -Dversion=1 -Dfile=ss.jar
mvn install:install-file -DgroupId=com.rsa -DartifactId=jsafe -Dpackaging=jar -Dversion=3.5 -Dfile=jsafe.jar
```
After that, the mvn targets should work as you generally expect them to.

TIP: Use mvn -o to build in offline mode after you've done an initial mvn install.  The -o option
     forces m2 to use jars from your local repository instead of the internet, which saves tons
     of time.

Unit Tests
----------
To pass the unit tests you will need to set up a test repository to run against.
You will also need to configure your settings.xml file (found in $HOME/.m2/ on
unix and %USERPROFILE%\.m2\ on windows).  Here are some instructions for setting
up both.

## Repository setup:

1. Create a repository
       * My repository is named "Hudson Plugin Test"
2. Add a file to the repository
       *  My test file is named "testfile.txt"
3. Modify test file and check in the changes
2. Label the repository with the label name "hudsonTestLabelBefore"
3. Modify "testfile.txt" again and check in the changes
4. Label the repository with the label name "hudsonTestLabel"
5. Modify "testfile.txt" again and check in the changes
6. Label the repository with the label name "hudsonTestLabelAfter"
7. Create a promotion state called "hudsonPromotionState" and point it at "hudsonTestLabel"

sample settings.xml:
```xml
<settings>
    <profiles>
        <profile>
        <id>hudson</id>
            <properties>
                <test.starteam.hostname></test.starteam.hostname>
                <test.starteam.hostport></test.starteam.hostport>
                <test.starteam.projectname>Hudson Plugin Test</test.starteam.projectname>
                <test.starteam.viewname>Hudson Plugin Test</test.starteam.viewname>
                <test.starteam.foldername>Hudson Plugin Test</test.starteam.foldername>
                <test.starteam.username></test.starteam.username>
                <test.starteam.password></test.starteam.password>
                <test.starteam.labelname>hudsonTestLabel</test.starteam.labelname>
                <test.starteam.promotionname>hudsonPromotionState</test.starteam.promotionname>
                <test.starteam.changedate></test.starteam.changedate>
                <test.starteam.dateinpast></test.starteam.dateinpast>
            </properties>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>hudson</activeProfile>
      </activeProfiles>
      <pluginGroups>
        <pluginGroup>org.jvnet.hudson.tools</pluginGroup>
    </pluginGroups>
</settings>
```
* Set changedate to the time of hudsonTestLabelAfter or later.
* Set dateinpast to the time of the first change to testfile.txt.

Known issues
------------
When developing with eclipse a situation can arise where
StarTeamSCMTest:testConfigRoundTrip fails with a NullSCM error.  I'm not sure
what causes this but the problem seems to be fixable by cleaning and building
on the command line.

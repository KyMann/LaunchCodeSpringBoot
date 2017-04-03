# LaunchCodeSpringBoot
In this workshop, we will add [OAuth2](https://tools.ietf.org/html/rfc6749) to [the cheese mvc app](https://github.com/LaunchCodeEducation/cheese-mvc) and push it to Pivotal Cloud Foundry


Prerequisites
=============

-   [Java 1.8 SDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

-   [Spring Tool Suite](https://spring.io/tools) or [IntelliJ](https://www.jetbrains.com/idea/) or [Eclipse](https://eclipse.org/downloads/)

-   [CF CLI](https://github.com/cloudfoundry/cli/releases)

-   [Curl](http://curl.haxx.se/) (Optional)

-   Git CLI for [Windows](https://github.com/git-for-windows/git/releases/download/v2.9.0.windows.1/Git-2.9.0-64-bit.exe) or Git from [github.com](https://desktop.github.com/) (Optional)

[Spring Boot](:https://projects.spring.io/spring-boot/) takes an opinionated view of building production-ready Spring applications. Favoring convention over configuration, it is designed to get you up and running as quickly as possible.

Spring Boot features include:

-   Creates a stand alone runnable jar file which includes everything required to run it

-   Ability to embed Tomcat, Jetty or Undertow making deployment much easier

-   Provide an oopinionated starter dependencies to simplify your configuration

-   Automatically configures Spring whenever possible

-   Provides production ready features such as metrics, health checks and externalized configuration

-   No code generation or XML configuration required

Part 1: Add Single Sign on with Facebook to [the cheese mvc app](https://github.com/LaunchCodeEducation/cheese-mvc)
===================
In this section we put Facebook for authentication on [the cheese mvc app](https://github.com/LaunchCodeEducation/cheese-mvc). This will be quite easy if we take advantage of the autoconfiguration features in Spring Boot.

Securing the Application

To make the application secure we just need to add Spring Security as a dependency. If we do that the default will be to secure it with HTTP Basic, so since we want to do a "social" login (delegate to Facebook), we add the Spring Security OAuth2 dependency as well:

Add the following lines in build.gradle:
	compile('org.springframework.boot:spring-boot-starter-security')
	compile('org.springframework.security.oauth:spring-security-oauth2')

To make the link to Facebook we need an @EnableOAuth2Sso annotation on our main class:
@SpringBootApplication
@EnableOAuth2Sso
public class CheeseMvcApplication {

	public static void main(String[] args) {
		SpringApplication.run(CheeseMvcApplication.class, args);
	}
}

and some configuration (converting application.properties to YAML for better readability):

application.yml

security:
  oauth2:
    client:
      clientId: 233668646673605
      clientSecret: 33b17e044ee6a4fa383f46ec6e28ea1d
      accessTokenUri: https://graph.facebook.com/oauth/access_token
      userAuthorizationUri: https://www.facebook.com/dialog/oauth
      tokenName: oauth_token
      authenticationScheme: query
      clientAuthenticationScheme: form
    resource:
      userInfoUri: https://graph.facebook.com/me
      
The configuration refers to a client app registered with Facebook in their [developers](https://developers.facebook.com/) site, in which you have to supply a registered redirect (home page) for the app. This one is registered to "localhost:8080" so it only works in an app running on that address.

With that change you can run the app again and visit the cheese page at http://localhost:8080/cheese. Instead of the cheese page you should be redirected to login with Facebook. If you do that, and accept any authorizations you are asked to make, you will be redirected back to the local app and the cheese page will be visible. If you stay logged into Facebook, you won’t have to re-authenticate with this local app, even if you open it in a fresh browser with no cookies and no cached data. (That’s what Single Sign On means.)

if you are working through this section with the sample application, be sure to clear your browser cache of cookies and HTTP Basic credentials.

What Just Happened?

The app you just wrote, in OAuth2 terms, is a Client Application and it uses the authorization code grant to obtain an access token from Facebook (the Authorization Server). It then uses the access token to ask Facebook for some personal details (only what you permitted it to do), including your login ID and your name. In this phase facebook is acting as a Resource Server, decoding the token that you send and checking it gives the app permission to access the user’s details. If that process is successful the app inserts the user details into the Spring Security context so that you are authenticated.

Part 2: Actuator
----------------

Remember earlier, when we selected dependencies for this project? We chose the Actuator and Actuator Docs dependencies. In this section, we’ll explore what Actuators are and how to use them.

One thing that all applicaitons have in common is the need to monitor and manage them.

### Actuator Endpoints

Spring Boot includes a number of built-in actuator endpoints that enable you to montior and interact with your application. Most endpoints are exposed via HTTP although other methods are available.

The most common endpoints are shown below:

-   health - Lists application health information

-   beans - Displays a list of all Spring Beans

-   info - Displays arbitrary application information

It’s important to note that not all endpoints are always on due to potential security concerns. Later on, we’ll change the behavior and enable endpoints.

For a list of all of the available endpoints see this [link](http://docs.spring.io/spring-boot/docs/1.5.1.RELEASE/reference/htmlsingle/#production-ready-endpoints)

1.  What happens when you go to your application’s /health endpoint in a browser? (e.g. [http://&lt;your](http://<your) application URL goes here&gt;/health)

2.  What happens when you go to your application’s /docs endpoint?

3.  What happens when you go to your application’s /info endpoint?

4.  What happens when you go to your application’s /autoconfig endpoint?

5.  What happens when you go to your application’s /beans endpoint?

#### Unlocking Protected Endpoints

In Spring Boot 1.5 and above, all actuator endpoints, with the exception of /health and /info are secured by default. Let’s make a change to our application to make the other endpoints visible.

1.  Edit the application.properties file. It is located in the src/main/resources folder.

2.  Add the following line to the file

        endpoints.sensitive=false

3.  Save the file

4.  Build the application jar file

    On a Mac run the following commands:

        $ cd <location of your project>
        $ ./gradlew clean build -x test

    On Windows run the following commands

<!-- -->

    > cd <location of your project>
    > gradlew clean build -x test

+ . Push the application to Pivotal Cloud Foundry

+

    $ cf push

Let’s hit a few of the endpoints that gave errors before.

1.  What happens when you visit the /beans endpoint?

2.  What happens when you visit the /autoconfig endpoint?

Try a few more of the endpoints and become familiar with the type of information they provide. Note that the output is in JSON format so using a JSON pretty printer like [this](http://jsonprettyprint.com) one is very helpful to make the output more readible.

#### Customize actuator behaviors

Customize HealthIndicator()

Part 3: Override property values without changing the jar file
===================
-   Traditional way: change properties file
-   Change environement variables
-   Change Java command line argument

CHALLENGES
Set both the environment variable and the command line argument and see what message you get.


Part 4: Running on Pivotal Cloud Foundry
===================

Now that the application has all the necessary features completed, it is time to push to Pivotal Cloud Foundry.

### Pushing to Pivotal Cloud Foundry

Before we deploy to cloud foundry there are a few things that need to occur.

1.  If you haven’t already, download the latest release of the Cloud Foundry CLI from [CF CLI](https://github.com/cloudfoundry/cli/releases) for your operating system and install it. 

2.  Sign up on [Pivotal Web Services](https://run.pivotal.io). No credit card needed. 

3.  In your cli, set the API target for the CLI: (this information will be provided to you in the workshop)

        $ cf api https://api.sys.cloud.rick-ross.com --skip-ssl-validation

4.  Follow the prompts, using the username & password you used to sign up PWS.

5.  Build the application jar file

    On a **Mac** use the following commands

        $ cd <location of your project>
        $ ./gradlew clean build -x test

    On **Windows** use the following commands

        > cd <location of your project>
        > gradlew.bat clean build -x test

    This creates a self-contained Jar file for the application in the *build/libs* folder. As an alternative, you can create the jar file within your IDE. For the purposes of this example, it is assumed that the location of the jar file is in the *build/libs* folder.

    Notice the **-x test** argument which tells Gradle that it should not run the unit tests. The reason we are using specifying this is because running the tests locally will fail because it will not be able to locate a local Redis repository.

6.  Push the application using the following command line

        $ cf push cheese --no-start --random-route -p build/libs/Spring-Person-0.0.1-SNAPSHOT.jar
        Creating app spring-person in org pivotal / space development as rross@pivotal.io...
        OK

        Creating route spring-person-commemoratory-isogeny.app.cloud.rick-ross.com...
        OK

        Binding spring-person-commemoratory-isogeny.app.cloud.rick-ross.com to spring-person...
        OK

        Uploading spring-person...
        Uploading app files from: /var/folders/mw/n4bhxvfn7wb4dw9rz8kznwcw0000gp/T/unzipped-app029402170
        Uploading 24.4M, 187 files
        Done uploading
        OK

    This command uploads the application to Pivotal Cloud Foundry, and does not start it because we still need to set up a Redis service.

7.  Browse the Marketplace

        $ cf marketplace
        Getting services from marketplace in org pivotal / space development as rross@pivotal.io...
        OK

        service                       plans                     description
        app-autoscaler                standard                  Scales bound applications in response to load (beta)
        p-circuit-breaker-dashboard   standard                  Circuit Breaker Dashboard for Spring Cloud Applications
        p-config-server               standard                  Config Server for Spring Cloud Applications
        p-mysql                       100mb                     MySQL databases on demand
        p-rabbitmq                    standard                  RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.
        p-redis                       shared-vm, dedicated-vm   Redis service to provide a key-value store
        p-service-registry            standard                  Service Registry for Spring Cloud Applications

        TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.

    Notice that there is a Redis service we can use. It is called "p-redis" and there are two plans: dedicated-vm and shared-vm.

8.  Create a Redis service using the shared-vm plan

        $ cf create-service p-redis shared-vm SpringPersonRedis
        OK

9.  Bind the application to this service

        $ cf bind-service spring-person SpringPersonRedis
        OK
        TIP: Use 'cf restage spring-person' to ensure your env variable changes take effect

10. Start the application

        $ cf start spring-person
        Starting app spring-person in org pivotal / space development as rross@pivotal.io...
        Downloading binary_buildpack...
        Downloading ruby_buildpack...
        Downloading python_buildpack...
        Downloading nodejs_buildpack...
        Downloading go_buildpack...
        Downloaded ruby_buildpack
        Downloading staticfile_buildpack...
        Downloaded binary_buildpack
        Downloading java_buildpack_offline...
        Downloaded nodejs_buildpack
        Downloaded go_buildpack
        Downloading php_buildpack...
        Downloaded python_buildpack
        Downloading dotnet_core_buildpack...
        Downloaded staticfile_buildpack
        Downloaded dotnet_core_buildpack
        Downloaded php_buildpack
        Downloaded java_buildpack_offline
        Creating container
        Successfully created container
        Downloading app package...
        Downloaded app package (37.3M)
        Staging...
        -----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
        -----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
               Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.1s)
        -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
               Memory Settings: -XX:MetaspaceSize=104857K -XX:MaxMetaspaceSize=104857K -Xss349K -Xmx681574K -Xms681574K
        -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
        Exit status 0
        Staging complete
        Uploading droplet, build artifacts cache...
        Uploading build artifacts cache...
        Uploading droplet...
        Uploaded build artifacts cache (109B)
        Uploaded droplet (82.4M)
        Uploading complete
        Destroying container
        Successfully destroyed container

        0 of 1 instances running, 1 starting
        0 of 1 instances running, 1 starting
        0 of 1 instances running, 1 starting
        1 of 1 instances running

        App started


        OK

        App spring-person was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher`

        Showing health and status for app spring-person in org pivotal / space development as rross@pivotal.io...
        OK

        requested state: started
        instances: 1/1
        usage: 1G x 1 instances
        urls: spring-person-heterochromatic-eelgrass.app.cloud.rick-ross.com
        last uploaded: Mon Feb 13 21:41:03 UTC 2017
        stack: cflinuxfs2
        buildpack: java-buildpack=v3.10-offline-https://github.com/cloudfoundry/java-buildpack.git#193d6b7 java-main open-jdk-like-jre=1.8.0_111 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10.0_RELEASE

             state     since                    cpu    memory       disk         details
        #0   running   2017-02-13 04:42:12 PM   0.0%   287M of 1G   165M of 1G

11. Open a browser and go to the URL indicated in the urls: line above, with "/persons" appended to the end of it. In this case the url is <https://spring-person-heterochromatic-eelgrass.app.cloud.rick-ross.com/persons>

    ![running on pcf](images/running-on-pcf.png)

Now we have an application that runs on Pivotal Cloud Foundry.

Creating a Manifest
-------------------

To make it easier to push updates to Pivotal Cloud Foundry, let’s create a manifest file.

1.  Create a file called manifest.yml and put it in the same folder that contains the pom.xml file.

2.  Edit the contents of this file to contain the following:

        ---
        applications:
        - name: spring-person
          memory: 1G
          random-route: true
          path: build/libs/Spring-Person-0.0.1-SNAPSHOT.jar
          services:
           - SpringPersonRedis

    Note that the Name of the Service needs to match the service you created previously. In this case it is *SpringPersonRedis*.

3.  Save the file

4.  Push the application again this time with no arguments

        $ cf push

5.  Open a browser and navigate to the /persons URL to verify the applicaiton is working



### Actuator Integration with Pivotal Cloud Foundry

Starting in Pivotal Cloud Foundry 1.9 and above, the Pivotal Apps Manager provides additional functionality for Spring Boot applications 1.5 and higher. In order to take advantage of these capabilities, let’s take a look at the application details in Apps Manager before we explore the additional capabilities.

1.  Open the Apps Manager in a browser

2.  Navigate to the Org and Space you are deploying the application to

3.  Click on spring-person application and the following screen appears:

    ![app manager](images/app-manager.png)

4.  Navigate into the Settings tab and notice the entries that are listed

    -   App Name

    -   Info

        ![spring person settings](images/spring-person-settings.png)

#### Adding Git Information (Optional)

This step requires that you have installed git on your computer. You will need to be able to create a local repository in order for this step to work.

##### For Maven Projects follow these instructions:

1.  Edit the pom.xml file. It is located in the root folder of the project.

2.  Navigate down to the plugin section. Add the following snippet below the existing spring-boot-maven-plugin

                    <plugin>
                        <groupId>pl.project13.maven</groupId>
                        <artifactId>git-commit-id-plugin</artifactId>
                    </plugin>

3.  Save the file

#### For Gradle Projects follow these instructions:

1.  Edit the build.gradle file. It is located in the root folder of the project.

2.  Navigate down to the plugin section. Add the following snippet before the apply plugin section

        plugins {
            id "com.gorylenko.gradle-git-properties" version "1.4.17"
        }

3.  Save the file

##### Editing the application.properties File

1.  Edit the application.properties file. It is located in the src/main/resources folder.

2.  Add the following line to the end of the file

        management.info.git.mode=full

These changes that we added to the project adds Git information to the /info endpoint. For additional details visit this [link](http://docs.pivotal.io/pivotalcf/1-9/console/spring-boot-actuators.html#git-info)

By adding these configuration changes (specifically the git commit plugin) you are indicating that your application has a git repository. Since we have started from scratch, let’s create a git repository.

Note: Failure to create a git repository will result in a build error.

1.  Open up a Command prompt or terminal window

2.  Change directories to the location of your application

3.  Run the following Git commands to initialize, add files and commit to the local repository

        $ git init
        $ git add .gitignore manifest.yml build.gradle gradlew gradlew.bat src/
        $ git commit -m 'Initial commit'

#### Adding Additional Information into Apps Manager

Let’s make a few minor changes to our application to expose additional details.

#### For Maven Projects follow these instructions

1.  Edit the pom.xml file. It is located in the root folder of the project

2.  Navigate down to the plugin section. Add the following snippet below the artifactId for spring-boot-maven-plugin

                <executions>
                            <execution>
                                <goals>
                                    <goal>build-info</goal>
                                </goals>
                            </execution>
                        </executions>

    The full section should look like this without the git entry.

            <build>
                <plugins>
                    <plugin>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-maven-plugin</artifactId>
                        <executions>
                                        <execution>
                                            <goals>
                                                    <goal>build-info</goal>
                                            </goals>
                                        </execution>
                                </executions>
                    </plugin>
                </plugins>
            </build>

    With the **optional** *Git Information*, this section looks like this

            <build>
                <plugins>
                    <plugin>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-maven-plugin</artifactId>
                        <executions>
                                        <execution>
                                            <goals>
                                                    <goal>build-info</goal>
                                            </goals>
                                        </execution>
                                </executions>
                    </plugin>
                    <plugin>
                        <groupId>pl.project13.maven</groupId>
                        <artifactId>git-commit-id-plugin</artifactId>
                    </plugin>
                </plugins>
            </build>

3.  Save the file

#### For Gradle Projects follow these instructions

1.  Edit the build.gradle file. It is located in the root folder of the project

2.  Add the following snippet before the dependencies section

        springBoot  {
            buildInfo()
        }

#### Editing the application.properties file

1.  Edit the application.properties file. It is located in the src/main/resources folder.

2.  Add the following line to the end of the file

        management.cloudfoundry.skip-ssl-validation=true
        info.app.version.java=@java.version@
        info.app.version.spring=@spring.version@

    The first line is only necessary if your Pivotal Cloud Foundry instance is using a self-signed certificate.

    You can also include information from Git which we won’t do in this workshop. For additional details see this [link](http://docs.pivotal.io/pivotalcf/1-9/console/spring-boot-actuators.html#git-info)

3.  Save the file

4.  Build the application jar file

    On a **Mac** use the following commands:

        $ cd <location of your project>
        $ ./gradlew clean build -x test

    On **Windows** use the following commands:

        > cd <location of your project>
        > gradlew.bat clean build -x test

5.  Push the application to Pivotal Cloud Foundry

        $ cf push

6.  Go back to the Apps Manager, navigate to the Space and click on spring-person.

7.  Click the View App link

8.  Accept the SSL Security Warnings

9.  Go back to Apps Manager and reload the page

    ![app manager spring with git](images/app-manager-spring-with-git.png)

    Notice the Spring Logo to the left of the application name. This indicates that Apps Manager has recognized this as a Spring Boot application.

    If you followed the optional Git Information steps above, you’ll see at the top right, a Git Commit ID

    Expanding the arrow next to the left of each application instance reveals the health check details, showing you details like the Redis health along with other basic metrics.

10. Navigate to the Settings tab

    ![app manager settings spring](images/app-manager-settings-spring.png)

    Notice the additional entries for Git and Spring Boot information where you can view the JSON

    Note that currently some information is only shown if the Git information has been included.




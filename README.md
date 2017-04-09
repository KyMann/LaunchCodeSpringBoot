# LaunchCodeSpringBoot
In this workshop, we will push [bootifulApplication](https://github.com/jennymclaughlin/bootifulApplications) to [Pivotal Web Services](http://run.pivotal.io) . We will also add [OAuth2](https://tools.ietf.org/html/rfc6749) to [bootifulApplication](https://github.com/jennymclaughlin/bootifulApplications).


Prerequisites
=============

-   [Java 1.8 SDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

-   [Spring Tool Suite](https://spring.io/tools) or [IntelliJ](https://www.jetbrains.com/idea/) or [Eclipse](https://eclipse.org/downloads/)

-   [CF CLI](https://github.com/cloudfoundry/cli/releases)

-   Git CLI for [Windows](https://github.com/git-for-windows/git/releases/download/v2.9.0.windows.1/Git-2.9.0-64-bit.exe) or Git from [github.com](https://desktop.github.com/) (Optional)

[Spring Boot](:https://projects.spring.io/spring-boot/) takes an opinionated view of building production-ready Spring applications. Favoring convention over configuration, it is designed to get you up and running as quickly as possible.

Spring Boot features include:

-   Creates a stand alone runnable jar file which includes everything required to run it

-   Ability to embed Tomcat, Jetty or Undertow making deployment much easier

-   Provide an oopinionated starter dependencies to simplify your configuration

-   Automatically configures Spring whenever possible

-   Provides production ready features such as metrics, health checks and externalized configuration

-   No code generation or XML configuration required

Part 1: Running on Pivotal Cloud Foundry
===================

### Pushing to Pivotal Cloud Foundry

Before we deploy to cloud foundry there are a few things that need to occur.

1.  If you haven’t already, download the latest release of the Cloud Foundry CLI from [CF CLI](https://github.com/cloudfoundry/cli/releases) for your operating system and install it. 

2.  Sign up on [Pivotal Web Services](https://run.pivotal.io). No credit card needed. 

3.  In your cli, set the API target for the CLI: (this information will be provided to you in the workshop)

        $ cf login -a api.run.pivotal.io

4.  Follow the prompts, using the username & password you used to sign up PWS.

5.  Build the application jar file

        $ cd <location of your project>
        $ mvn clean package


    This creates a self-contained Jar file that includes a tomcat servlet engine and all the necessary resources needed for this application.

6.  Push the application using the following command line

        $ cf push reservation --random-route -p target/reservation-service-0.0.1-SNAPSHOT.jar

7. Scale the application by changing the number of instances (scale out/in) or memory/disk space (scale up/down).
	

### Actuator Endpoints

Spring Boot includes a number of built-in actuator endpoints that enable you to montior and interact with your application. Most endpoints are exposed via HTTP although other methods are available.

The most common endpoints are shown below:

-   health - Lists application health information

-   beans - Displays a list of all Spring Beans

-   info - Displays arbitrary application information

For a list of all of the available endpoints see this [link](http://docs.spring.io/spring-boot/docs/1.5.1.RELEASE/reference/htmlsingle/#production-ready-endpoints)

1.  What happens when you go to your application’s /health endpoint in a browser? (e.g. [http://&lt;your](http://<your) application URL goes here&gt;/health)

2.  What happens when you go to your application’s /docs endpoint?

3.  What happens when you go to your application’s /info endpoint?

4.  What happens when you go to your application’s /autoconfig endpoint?

5.  What happens when you go to your application’s /beans endpoint?

Part 2: Add Single Sign on with Facebook to [bootifulApplication](https://github.com/jennymclaughlin/bootifulApplications)
===================
In this section we put Facebook for authentication on [bootifulApplication](https://github.com/jennymclaughlin/bootifulApplications). This will be quite easy if we take advantage of the autoconfiguration features in Spring Boot.

Securing the Application

To make the application secure we need to add Spring Security as a dependency. If we do that the default will be to secure it with HTTP Basic, so since we want to do a "social" login (delegate to Facebook), we add the Spring Security OAuth2 dependency as well:

Add the following dependencies in pom.xml:

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-security</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.security.oauth</groupId>
		<artifactId>spring-security-oauth2</artifactId>
	</dependency>

To make the link to Facebook we need an @EnableOAuth2Sso annotation on our main class:
@SpringBootApplication
@EnableOAuth2Sso
public class ReservationServiceApplication {

and some configuration (using application.yml for better readability):

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

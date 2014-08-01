---
title: User Account and Authentication (UAA) Server
---

The UAA is the identity management service for Cloud Foundry.
Its primary role is as an OAuth2 provider, issuing tokens for client applications to use when they act on behalf of Cloud Foundry users.
It can also authenticate users with their Cloud Foundry credentials, and can act as an SSO service using those credentials (or others).
It has endpoints for managing user accounts and for registering OAuth2 clients, as well as various other management functions.

## <a id='quickstart'></a>Quick Start ##

Fetch and install the UAA from Git:

<pre class="terminal">
$ git clone git://github.com/cloudfoundry/uaa.git
$ cd uaa
$ mvn install
</pre>

Each module has a `mvn tomcat:run` target to run individually.
Alternatively, you could import the modules as projects into STS (2.8.0 or better).
The apps all work together and run on the same port (8080) with endpoints `/uaa`, `/app` and `/api`.

You will need Maven 3.0.4 or newer.

### <a id='deploy'></a>Deploy to Cloud Foundry ###

You can also build the app and push it to Cloud Foundry:

<pre class="terminal">
$ mvn package install
$ cf push myuaa --path uaa/target
</pre>

If you do that, choose a unique application id, not `myuaa`.

### <a id='demo-local'></a>Local Command Line Usage ###

First run the UAA server as described above:

<pre class="terminal">
$ cd uaa
$ mvn tomcat:run
</pre>

Then start another terminal and, from the project base directory, ask the login endpoint to tell you about the system:

<pre class="terminal">
$ curl -H "Accept: application/json" localhost:8080/uaa/login
{
  "timestamp":"2012-03-28T18:25:49+0100",
  "commit_id":"111274e",
  "prompts":{"username":["text","Email"],
    "password":["password","Password"]
  }
}
</pre>

Then you can try logging in with the UAA Ruby gem. Make sure you have Ruby 1.9, then:

<pre class="terminal">
$ gem install cf-uaac
$ uaac target http://localhost:8080/uaa
$ uaac token get marissa koala
</pre>

If you do not specify username and password, you will be prompted to supply them.

This authenticates and obtains an access token from the server using the OAuth2 implicit grant, similar to the approach intended for a client like `cf`.
The token is stored in `~/.uaac.yml`.
Open that file to obtain the access token for your `cf` target, or use `--verbose` on the login command line above to see it in the command shell.

Then you can login as a resource server and retrieve the token details:

<pre class="terminal">
$ uaac target http://localhost:8080/uaa
$ uaac token decode [token-value-from-above]
</pre>

You should see your username and the client id of the original token grant on stdout:

~~~
exp: 1355348409
user_name: marissa
scope: cloud_controller.read openid password.write scim.userids tokens.read tokens.write
email: marissa@test.org
aud: scim tokens openid cloud_controller password
jti: ea2fac72-3f51-4c8f-a7a6-5ffc117af542
user_id: ba14fea0-9d87-4f0c-b59e-32aaa8eb1434
client_id: cf
~~~

### <a id='demo-hosted'></a>Remote Command Line Usage ###

The command line example in the previous section should work against a UAA running on the hosted Pivotal CF instance, although token encoding is unnecessary as you will not have the client secret.

In this case, there is no need to run a local uaa server.
Ask the external login endpoint to tell you about the system:

<pre class="terminal">
$ curl -H "Accept: application/json" uaa.run.pivotal.io/login
{
  "prompts":{"username":["text","Email"],
    "password":["password","Password"]
  }
}
</pre>

You can then try logging in with the UAA Ruby gem. Make sure you have Ruby 1.9, then:

<pre class="terminal">
$ gem install cf-uaac
$ uaac target uaa.run.pivotal.io
$ uaac token get [yourusername] [yourpassword]
</pre>

If you do not specify a username and password, you will be prompted to supply them.

This authenticates and obtains an access token from the server using the OAuth2 implicit grant, which is the same that a client like `cf` uses.

## <a id='integration'></a>Integration tests ##

With all apps deployed into a running server on port 8080, the tests will include integration tests (a check is done before each test that the app is running).
You can deploy them in your IDE or using the command line with `mvn tomcat:run`, then run the tests as normal.

For individual modules, or for the whole project, you can also run integration tests and the server from the command line:

<pre class="terminal">
$ mvn test -P integration
</pre>

This might first require an initial `mvn install` from the parent directory to get the WARs in your local repo.

To make the tests work in various environments, you can modify the configuration of the server and the tests (e.g. the admin client) using a variety of mechanisms.
The simplest is to provide additional Maven profiles on the command line:

<pre class="terminal">
$ (cd uaa; mvn test -P vcap)
</pre>

This runs the integration tests against a UAA server running in a local vcap, so for example the service URL is set to `uaa.vcap.me` (by default).
There are several Maven profiles to play with, and you can use them to run the server, the tests, or both:

* `local`: runs the server on the ROOT context `http://localhost:8080/`

* `vcap`: also runs the server on the ROOT context and points the tests at `uaa.vcap.me`.

* `devuaa`: points the tests at `http://devuaa.cloudfoundry.com` (an instance of UAA deployed on cloudfoundry) *NB this information is currently outdated*

All these profiles set the `CLOUD_FOUNDRY_CONFIG_PATH` to pick up a `uaa.yml` and (if appropriate) set the context root for running the server.

### <a id='integration-tests'></a>Integration Tests ###

There is a simple cucumber feature spec (`--tag @uaa`) to verify that the UAA server is there.
There is also a rake task to launch the integration tests from the `uaa` submodule in `vcap`.
Typical usage for a local (`uaa.vcap.me`) instance:

<pre class="terminal">
$ cd vcap/tests
$ rake bvt:run_uaa
</pre>

You can change the most common important settings with environment variables (see below), or with a custom `uaa.yml`.

<p class="note"><strong>Note</strong>: You cannot use <code>MAVEN_OPTS</code> to set JVM system properties for the tests, but you can use it to set memory limits for the process, etc.</p>

### <a id='custom-configuration'></a>Custom YAML Configuration ###

To modify the runtime parameters, you can provide a `uaa.yml`:

<pre class="terminal">
$ cat > /tmp/uaa.yml
uaa:
  host: uaa.appcloud21.dev.mozycloud
  test:
    username: dev@cloudfoundry.org # defaults to vcap_tester@vmware.com
    password: changeme
    email: dev@cloudfoundry.org
</pre>

then from `vcap-tests`:

<pre class="terminal">
$ CLOUD_FOUNDRY_CONFIG_PATH=/tmp rake bvt:run_uaa
</pre>

or from `uaa/uaa`:

<pre class="terminal">
$ CLOUD_FOUNDRY_CONFIG_PATH=/tmp mvn test
</pre>

The integration tests look for a YAML file in the following locations (later entries override earlier ones), and the webapp does the same when it starts up, so you can use the same config file for both:

```yaml
classpath:uaa.yml
file:${CLOUD_FOUNDRY_CONFIG_PATH}/uaa.yml
file:${UAA_CONFIG_FILE}
${UAA_CONFIG_URL}
```

### <a id='maven'></a>Using Maven with Cloud Foundry or VCAP ###

To test against a vcap instance, use the Maven profile `vcap`. This profile switches off some of the tests that create random client and user accounts:

<pre class="terminal">
$ (cd uaa; mvn test -P vcap)
</pre>

To change the target server, set `VCAP_BVT_TARGET` (the tests prefix it with `uaa.` to form the server url):

<pre class="terminal">
$ VCAP_BVT_TARGET=appcloud21.dev.mozycloud mvn test -P vcap
</pre>

You can also override some of the other most important default settings using environment variables.
The defaults come from `uaa.yml`, but tests will first search in an environment variable:

* `UAA_ADMIN_CLIENT_ID` the client id for bootstrapping client registrations needed for the rest of the tests.

* `UAA_ADMIN_CLIENT_SECRET` the client secret for bootstrapping client registrations

You can individually override all other settings from `uaa.yml` as system properties.
To do this in an IDE, use the IDE features that allow you to modify the JVM in test runs.
With Maven, use the `argLine` property to get settings passed onto the test JVM:

<pre class="terminal">
$ mvn -DargLine=-Duaa.test.username=foo test
</pre>

This creates an account with `userName=foo` for testing (instead of using the default setting from `uaa.yml`).

If you prefer environment variables to system properties, you can use a custom `uaa.yml` with placeholders for your environment variables:

```yaml
uaa:
  test:
    username: ${UAA_TEST_USERNAME:marissa}
```

This looks for an environment variable (or system property) `UAA_TEST_USERNAME` before defaulting to `marissa`.
This is the trick used to expose `UAA_ADMIN_CLIENT_SECRET` etc. in the standard configuration.

### <a id='maven-test'></a>Test with PostgreSQL or MySQL ##

The default UAA unit tests (`mvn test`) use hsqldb.

To run the unit tests using PostgreSQL:

<pre class="terminal">
$ SPRING_PROFILES_ACTIVE=test,postgresql CLOUD_FOUNDRY_CONFIG_PATH=src/test/resources/test/profiles/postgresql mvn test
</pre>

To run the unit tests using MySQL:

<pre class="terminal">
$ SPRING_PROFILES_ACTIVE=test,mysql CLOUD_FOUNDRY_CONFIG_PATH=src/test/resources/test/profiles/mysql mvn test
</pre>

The database configuration for the common and scim modules is located at `common/src/test/resources/(mysql|postgresql).properties` and `scim/src/test/resources/(mysql|postgresql).properties`.

## <a id='projects'></a>UAA Projects ##

There are actually several projects here: the main `uaa` server application, and some samples:

* `common` is a module containing a JAR with all the business logic.  It is used in
the webapps below.

* `uaa` is the actual UAA server. `uaa` provides an authentication service plus authorized delegation for back-end services and apps (by issuing OAuth2 access tokens).

* `gem` is a ruby gem (`cf-uaa-client`) for interacting with the UAA server.

* `api` (sample) is an OAuth2 resource service which returns a mock list of deployed apps. `api` is a service that provides resources that other applications might want to access on behalf of the resource owner (the end user).

* `app` (sample) is a user application that uses both of the above. `app` is a webapp that needs single sign-on and access to the `api` service on behalf of users.

* `login` (sample) is an application that performs authentication for the UAA acting as a back end service. `login` is where Cloud Foundry administrators set up their authentication sources, e.g. LDAP/AD, SAML, OpenID (Google etc.) or social. The `run.pivotal.io` platform uses a different implementation of the [login server](https://github.com/cloudfoundry/login-server).


## <a id='uaa-server'></a>UAA Server ##

The authentication service is `uaa`. It is a plain Spring MVC webapp.
Deploy as normal in Tomcat or your container of choice, or execute `mvn tomcat:run` to run it directly from `uaa` directory in the source tree (make sure the common jar is installed first using `mvn install` from the common subdirectory or from the top level directory).
With Maven, it listens on port 8080.

The UAA Server supports the APIs defined in the UAA-APIs document. To summarize:

1. The OAuth2 `/authorize` and `/token` endpoints

2. A `/login_info` endpoint to allow querying for required login prompts

3. A `/check_token` endpoint, to allow resource servers to obtain information about an access token submitted by an OAuth2 client.

4. SCIM user provisioning endpoint

5. OpenID connect endpoints to support authentication `/userinfo` and `/check_id` (todo). Implemented roughly enough to get it working (so `/app` authenticates here), but not to meet the spec.

Command line clients can perform authentication by submitting credentials directly to the `/authorize` endpoint.
There is an `ImplicitAccessTokenProvider` in Spring Security OAuth that can do the heavy lifting if your client is Java.

By default, `uaa` will launch with a context root `/uaa`.
There is a Maven profile `local` to launch with context root `/`, and another called `vcap` to launch at `/` with a PostgreSQL backend.

### <a id='configuration'></a>Configuration ###

There is a `uaa.yml` file in the application which provides defaults to the placeholders in the Spring XML.
Wherever you see `${placeholder.name}` in the XML, there is an opportunity to override it either by providing a System property (`-D` to JVM) with the same name, or a custom `uaa.yml` (as described above).

All passwords and client secrets in the config files are plain text, but they will be inserted into the UAA database encrypted with BCrypt.

### <a id='user-account-data'></a>User Account Data ###

The default is to use an in-memory RDBMS user store that is pre-populated with a single test users: `marissa` has password `koala`.

To use Postgresql for user data, activate one of the Spring profiles `hsqldb` or `postgresql`.

You can configure the active profiles in `uaa.yml` using:

```yaml
spring_profiles: postgresql
```

or by passing the `spring.profiles.active` parameter to the JVM. For example, to run with an embedded HSQL database:

<pre class="terminal>
$ mvn -Dspring.profiles.active=hsqldb tomcat:run
</pre>

Or to use PostgreSQL instead of HSQL:

<pre class="terminal>
$ mvn -Dspring.profiles.active=postgresql tomcat:run
</pre>

To bootstrap a microcloud type environment, you need an admin client; there is a database initializer component that inserts one.
If the default profile is active (i.e. not `postgresql`), there is also a `cf` client so that the gem login works out of the box.
You can override the default settings and add additional clients in `uaa.yml`:

```yaml
oauth:
  clients:
    admin:
      authorized-grant-types: client_credentials
      scope: read,write,password
      authorities: ROLE_CLIENT,ROLE_ADIN
      id: admin
      secret: adminclientsecret
      resource-ids: clients
```
You can use the admin client to create additional clients.
You will need a client with read/write access to the `scim` resource to create user accounts.
The integration tests take care of this automatically by inserting client and user accounts as necessary to make the tests work.

## <a id='samples'></a>Sample Applications ##

### <a id='api-application'></a>API Application ###

An example resource server.
It hosts a service that returns a list of mock applications under `/apps`.

Run it using `mvn tomcat:run` from the `api` directory once all other Tomcat processes have been shut down.
This deploys the app to a Tomcat manager on port 8080.

### <a id='app-application'></a>App Application ###

This is a user interface app (primarily aimed at browsers) that uses OpenId Connect for authentication (i.e. SSO) and OAuth2 for access grants.
It authenticates with the Auth service, then accesses resources in the API service.
Run it with `mvn tomcat:run` from the `app` directory once all other Tomcat processes have been shut down.

The application can operate in multiple different profiles according to the location (and presence) of the UAA server and the Login application.
By default, the application looks for a UAA on `localhost:8080/uaa`, but you can change this by setting an environment variable (or System property) called `UAA_PROFILE`.
In the application source code (`src/main/resources`), you will find multiple properties files pre-configured with different likely locations for those servers.
They are all in the form `application-<UAA_PROFILE>.properties`.
The naming convention adopted is that the `UAA_PROFILE` is `local` for the localhost deployment, `vcap` for a `vcap.me` deployment, and `staging` for a staging deployment.
The profile names are double-barreled (e.g. `local-vcap` when the login server is in a different location than the UAA server).

#### Use Cases ####

1. See all apps

        GET /app/apps

    Browser is redirected through a series of authentication and access grant steps (which could be slimmed down to implicit steps not requiring user at some point), and then the photos are shown.

2. See the currently logged in user details, a bag of attributes grabbed from the open id provider

        GET /app

### <a id='login-application'></a>Login Application ###

A user interface for authentication.
The UAA can also authenticate user accounts, but only if it manages them itself, and it only provides a basic UI.
You can brand and customize the login app for non-native authentication and for more complicated UI flows, like user registration and password reset.

The login application is itself an OAuth2 endpoint provider, but delegates those features to the UAA server.
Therefore, configuration for the login application consists of locating the UAA through its OAuth2 endpoint URLs and registering the login application itself as a client of the UAA.
There is a `login.yml` for the UAA locations, such as for a local `vcap` instance:

```yaml
uaa:
  url: http://uaa.vcap.me
  token:
    url: http://uaa.vcap.me/oauth/token
  login:
    url: http://uaa.vcap.me/login.do
```

There is also an environment variable (or Java System property), `LOGIN_SECRET`, for the client secret that the app uses when it authenticates itself with the UAA.
The login app is registered by default in the UAA only if there are no active Spring profiles.
In the UAA, the registration is located in the `oauth-clients.xml` config file:

```yaml
id: login
secret: loginsecret
authorized-grant-types: client_credentials
authorities: ROLE_LOGIN
resource-ids: oauth
```

#### Use Cases ####

1. Authenticate

        GET /login

    The sample app presents a form login interface for the backend UAA, and also an OpenID widget so the user can authenticate using Google credentials or others.

2. Approve OAuth2 token grant

        GET /oauth/authorize?client_id=app&response_type=code...

    Standard OAuth2 Authorization Endpoint. The UAA handles client credentials and all other features in the back end, and the login application is used to render the UI (see `access_confirmation.jsp`).

3. Obtain access token

        POST /oauth/token

    Standard OAuth2 Authorization Endpoint passed through to the UAA.

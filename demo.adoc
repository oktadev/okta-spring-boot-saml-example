:experimental:
:commandkey: &#8984;
:toc: macro
:source-highlighter: highlight.js

= Get Started with Spring Boot and SAML

One of my favorite Spring projects is Spring Security. In most cases, it simplifies web security to just a few lines of code. HTTP Basic, JDBC, JWT, OpenID Connect/OAuth 2.0, you name it&mdash;Spring Security does it!

You might notice I didn't mention SAML as an authentication type. That's because I don't recommend it. The specification for SAML 2.0 was published in March 2005, before smartphones or smart devices even existed. OpenID Connect (OIDC) is much easier for developers to use and understand. Using SAML in 2022 is like implementing a web service using WS-* instead of REST.

My recommendation: just use OIDC.

If you _must_ use SAML with Spring Boot, this screencast should make it quick and easy.

_Check the description below this video for links to the blog post and this demo script._

**Prerequisites**:

- https://adoptium.net/[Java 17]: I recommend using https://sdkman.io/[SDKMAN!] to manage and install multiple versions of Java.

toc::[]

[TIP]
====
The brackets at the end of some steps indicate the IntelliJ Live Templates to use. You can find the template definitions at https://github.com/mraible/idea-live-templates[mraible/idea-live-templates].

You can also expand the file names to see the full code.
====

**Fast Track**: https://github.com/oktadev/okta-spring-boot-saml-example[Clone the repo] and follow the instructions in its `README` to configure everything.

== Add a SAML application on Okta

. To begin, you'll need an Okta developer account. You can create one at https://developer.okta.com/signup[developer.okta.com/signup] or install the https://cli.okta.com[Okta CLI] and run `okta register`.

. Log in to your account and go to *Applications* > *Create App Integration*. Select *SAML 2.0* and click *Next*. Name your app something like `Spring Boot SAML` and click *Next*.

. Use the following settings:

* Single sign on URL: `\http://localhost:8080/login/saml2/sso/okta`
* Use this for Recipient URL and Destination URL: ✅ (the default)
* Audience URI: `\http://localhost:8080/saml2/service-provider-metadata/okta`

. Click *Next*. Select the following options:

* I'm an Okta customer adding an internal app
* This is an internal app that we have created

. Select *Finish*.

. Scroll down to the *SAML Signing Certificates* and go to *SHA-2* > *Actions* > *View IdP Metadata*. You can right-click and copy this menu item's link or open its URL. Copy the resulting link to your clipboard.

. Go to your app's *Assignment* tab and assign access to the *Everyone* group.

== Create a Spring Boot app with SAML support

. Create a Spring Boot app using https://start.spring.io[start.spring.io]. Select the following options:

* Project: *Gradle*
* Spring Boot: *3.0.6*
* Dependencies: *Spring Web*, *Spring Security*, *Thymeleaf*
+
You can also use https://start.spring.io/#!type=gradle-project&language=java&platformVersion=3.0.6&packaging=jar&jvmVersion=17&groupId=com.example&artifactId=demo&name=demo&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.demo&dependencies=web,security,thymeleaf[this URL] or HTTPie:
+
[source,shell]
----
https start.spring.io/starter.zip bootVersion==3.0.6 \
  dependencies==web,security,thymeleaf type==gradle-project \
  baseDir==spring-boot-saml | tar -xzvf -
----

. Add a `HomeController.java` to populate the authenticated user's information. [`saml-home`]
+
.`HomeController.java`
[%collapsible]
====
[source,java]
----
package com.example.demo;

import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.saml2.provider.service.authentication.Saml2AuthenticatedPrincipal;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HomeController {

    @RequestMapping("/")
    public String home(@AuthenticationPrincipal Saml2AuthenticatedPrincipal principal, Model model) {
        model.addAttribute("name", principal.getName());
        model.addAttribute("emailAddress", principal.getFirstAttribute("email"));
        model.addAttribute("userAttributes", principal.getAttributes());
        return "home";
    }

}
----
====

. Create a `src/main/resources/templates/home.html` file to render the user's information. [`saml-home-html`]
+
.`home.html`
[%collapsible]
====
[source,html]
----
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity6">
<head>
    <title>Spring Boot and SAML</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>

<h1>Welcome</h1>
<p>You are successfully logged in as <span sec:authentication="name"></span></p>
<p>Your email address is <span th:text="${emailAddress}"></span>.</p>
<p>Your authorities are <span sec:authentication="authorities"></span>.</p>
<h2>All Your Attributes</h2>
<dl th:each="userAttribute : ${userAttributes}">
    <dt th:text="${userAttribute.key}"></dt>
    <dd th:text="${userAttribute.value}"></dd>
</dl>

<form th:action="@{/logout}" method="post">
    <button id="logout" type="submit">Logout</button>
</form>

</body>
</html>
----
====

. Rename `application.properties` to `application.yml` and add SAML configuration for Spring Security. [`saml-config`]
+
.`application.yml`
[%collapsible]
====
[source,yaml]
----
spring:
  security:
    saml2:
      relyingparty:
        registration:
          okta:
            assertingparty:
              metadata-uri: <your-metadata-uri>
----
====

. Update `build.gradle` to add Spring Security's SAML dependency:
+
[source,groovy]
----
repositories {
    ...
    maven { url "https://build.shibboleth.net/nexus/content/repositories/releases/" }
}

dependencies {
    constraints {
        implementation "org.opensaml:opensaml-core:4.1.1"
        implementation "org.opensaml:opensaml-saml-api:4.1.1"
        implementation "org.opensaml:opensaml-saml-impl:4.1.1"
    }
    ...
    implementation 'org.springframework.security:spring-security-saml2-service-provider'
}
----

=== Run the app and authenticate

. Run your Spring Boot app from your IDE or using the command line:
+
[source,shell]
----
./gradlew bootRun
----

. Open `\http://localhost:8080` in your favorite browser and log in with the credentials you used to create your account.

. If you try to log out, it won't work. Let's fix that.

=== Add a logout feature

. Edit your application on Okta and navigate to *General* > *SAML Settings* > *Edit*.

. Continue to the *Configure SAML* step and *Show Advanced Settings*. Before you can enable single logout, you'll have to create and upload a certificate to sign the outgoing logout request.

. You can create a private key and certificate using OpenSSL. Answer at least one of the questions with a value, and it should work.
+
[source,shell]
----
openssl req -newkey rsa:2048 -nodes -keyout local.key -x509 -days 365 -out local.crt
----

. Copy the generated files to your app's `src/main/resources` directory. Configure `signing` and `singlelogout` in `application.yml`:
+
[source,yaml]
----
spring:
  security:
    saml2:
      relyingparty:
        registration:
          okta:
            assertingparty:
              ...
            signing:
              credentials:
                - private-key-location: classpath:local.key
                  certificate-location: classpath:local.crt
            singlelogout:
              binding: POST
              response-url: "{baseUrl}/logout/saml2/slo"
----

. Upload the `local.crt` to your Okta app. Select *Enable Single Logout* and use the following values:

* Single Logout URL: `\http://localhost:8080/logout/saml2/slo`
* SP Issuer: `\http://localhost:8080/saml2/service-provider-metadata/okta`

. Finish configuring your Okta app, restart your Spring Boot app, and logout should work.

=== Customize authorities with Spring Security SAML

You might notice when you log in, the resulting page shows you have a `ROLE_USER` authority. However, when you assigned users to the app, you gave access to `Everyone`. You can configure your SAML app on Okta to send a user's groups as an attribute. You can add other attributes like name and email too.

. Edit your Okta app's SAML settings and fill in the *Group Attribute Statements* section.

* Name: `groups`
* Name format: `Unspecified`
* Filter: `Matches regex` and use `.*` for the value

. Just above, you can add other attribute statements. For instance:
+
|===
|Name |Name format|Value

|`email`
|`Unspecified`
|`user.email`

|`firstName`
|`Unspecified`
|`user.firstName`

|`lastName`
|`Unspecified`
|`user.lastName`
|===

. *Save* these changes.

. Create a `SecurityConfiguration` class that overrides the default configuration and uses a converter to translate the values in the `groups` attribute into Spring Security authorities. [`saml-security-config`]
+
.`SecurityConfiguration.java`
[%collapsible]
====
[source,java]
----
package com.example.demo;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.saml2.provider.service.authentication.OpenSaml4AuthenticationProvider;
import org.springframework.security.saml2.provider.service.authentication.OpenSaml4AuthenticationProvider.ResponseToken;
import org.springframework.security.saml2.provider.service.authentication.Saml2AuthenticatedPrincipal;
import org.springframework.security.saml2.provider.service.authentication.Saml2Authentication;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {

    @Bean
    SecurityFilterChain configure(HttpSecurity http) throws Exception {

        OpenSaml4AuthenticationProvider authenticationProvider = new OpenSaml4AuthenticationProvider();
        authenticationProvider.setResponseAuthenticationConverter(groupsConverter());

        http.authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated())
            .saml2Login(saml2 -> saml2
                .authenticationManager(new ProviderManager(authenticationProvider)))
            .saml2Logout(withDefaults());

        return http.build();
    }

    private Converter<OpenSaml4AuthenticationProvider.ResponseToken, Saml2Authentication> groupsConverter() {

        Converter<ResponseToken, Saml2Authentication> delegate =
            OpenSaml4AuthenticationProvider.createDefaultResponseAuthenticationConverter();

        return (responseToken) -> {
            Saml2Authentication authentication = delegate.convert(responseToken);
            Saml2AuthenticatedPrincipal principal = (Saml2AuthenticatedPrincipal) authentication.getPrincipal();
            List<String> groups = principal.getAttribute("groups");
            Set<GrantedAuthority> authorities = new HashSet<>();
            if (groups != null) {
                groups.stream().map(SimpleGrantedAuthority::new).forEach(authorities::add);
            } else {
                authorities.addAll(authentication.getAuthorities());
            }
            return new Saml2Authentication(principal, authentication.getSaml2Response(), authorities);
        };
    }
}
----
====

. Restart your app and log in, you should see your user's groups as authorities.

== Add support for Auth0

. https://auth0.com/signup[Sign up for an Auth0 account] or https://auth0.com/api/auth/login?redirectTo=dashboard[log in] with your existing one. Navigate to *Applications* > *Create Application* > *Regular Web Applications* > *Create*.

. Select the *Settings* tab and change the name to `Spring Boot SAML`. Add `\http://localhost:8080/login/saml2/sso/auth0` as an *Allowed Callback URL*.

. Scroll to the bottom, expand *Advanced Settings*, and go to *Endpoints*. Copy the value of the *SAML Metadata URL*. Select *Save Changes*.

. If you configure your app to use the metadata URL, authentication will work, but you won't be able to log out. Scroll to the top of the page, select *Addons*, and enable SAML.

. Select the *Settings* tab and change the (commented) JSON to be as follows:
+
[source,json]
----
{
  "logout": {
    "callback": "http://localhost:8080/logout/saml2/slo",
    "slo_enabled": true
  }
}
----

. Scroll to the bottom and click *Enable*.

. Change your `application.yml` to use `auth0` instead of `okta` and copy your *SAML Metadata URL* into it.
+
[source,yaml]
----
spring:
  security:
    saml2:
      relyingparty:
        registration:
          auth0:
            assertingparty:
              metadata-uri: <your-auth0-metadata-uri>
----

. Restart your app, and you should be able to log in with Auth0!

. You might notice that the email and authorities are not calculated correctly. This is because the claim names have changed with Auth0. Update `SecurityConfiguration#groupsConverter()` to allow both Okta and Auth0 names for groups.
+
[source,java]
----
private Converter<OpenSaml4AuthenticationProvider.ResponseToken, Saml2Authentication> groupsConverter() {

    ...

    return (responseToken) -> {
        ...
        List<String> groups = principal.getAttribute("groups");
        // if groups is not preset, try Auth0 attribute name
        if (groups == null) {
            groups = principal.getAttribute("http://schemas.auth0.com/roles");
        }
        ...
    };
}
----

. To make Auth0 populate a user's groups, navigate to *Actions* > *Flows* and select *Login*. Create a new action named `Add Roles` and use the default trigger and runtime. Change the `onExecutePostLogin` handler to be as follows:
+
[source,js]
----
exports.onExecutePostLogin = async (event, api) => {
  if (event.authorization) {
    api.idToken.setCustomClaim('preferred_username', event.user.email);
    api.idToken.setCustomClaim(`roles`, event.authorization.roles);
    api.accessToken.setCustomClaim(`roles`, event.authorization.roles);
  }
}
----

. Deploy the action, add it to your login flow, and apply the changes.

. Modify `HomeController` to allow Auth0's email attribute name.
+
[source,java]
----
public class HomeController {

    @RequestMapping("/")
    public String home(@AuthenticationPrincipal Saml2AuthenticatedPrincipal principal, Model model) {
        model.addAttribute("name", principal.getName());
        String email = principal.getFirstAttribute("email");
        // if email is not preset, try Auth0 attribute name
        if (email == null) {
            email = principal.getFirstAttribute("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress");
        }
        model.addAttribute("emailAddress", email);
        model.addAttribute("userAttributes", principal.getAttributes());
        return "home";
    }

}
----

. Restart your app, log in, and everything should work as expected.

== Support Okta and Auth0

You can also support _both_ Okta and Auth0! Modify your `application.yml` to be as follows, and Spring Security will prompt you for which one to log in with. The `&name` and `*name` values are used to set and retrieve blocks of YAML to avoid repetition.

[source,yaml]
----
spring:
  security:
    saml2:
      relyingparty:
        registration:
          auth0:
            assertingparty:
              metadata-uri: <your-auth0-metadata-uri>
            signing:
              credentials: &signing-credentials
                - private-key-location: classpath:local.key
                  certificate-location: classpath:local.crt
            singlelogout: &logout-settings
              binding: POST
              response-url: "{baseUrl}/logout/saml2/slo"
          okta:
            assertingparty:
              metadata-uri: <your-okta-metadata-uri>
            signing:
              credentials: *signing-credentials
            singlelogout: *logout-settings
----

== Deploy to production

One quick way to see this app working in a production environment is to deploy it to Heroku. https://devcenter.heroku.com/articles/heroku-cli[Install the Heroku CLI] and create an account to begin. Then, follow the steps below to prepare and deploy your app.

. Create a new app on Heroku using `heroku create`.

. Create a `system.properties` file in the root directory of your app to force Java 17:
+
[source,properties]
----
java.runtime.version=17
----

. Create a `Procfile` that specifies how to run your app:
+
----
web: java -Xmx256m -jar build/libs/*.jar --server.port=$PORT
----

. Commit your changes and add Heroku as a remote:
+
----
git init
git add .
git commit -m "Spring Boot SAML example"
heroku git:remote -a <your-heroku-app-name>
----

. Set the Gradle task to build your app:
+
[source,shell]
----
heroku config:set GRADLE_TASK="bootJar"
----

. Deploy to production using Git:
+
[source,shell]
----
git push heroku main
----

. For everything to work, you'll need to update your Okta and Auth0 apps to use your Heroku app's URL in place of `\http://localhost:8080`, wherever applicable.

== Have fun with Spring Boot and Spring Security!

I hope you enjoyed this Spring Boot demo, and it helped you learn how to use Spring Security with SAML.

🧰 Find the source code on GitHub: https://github.com/oktadev/okta-spring-boot-saml-example[@oktadev/okta-spring-boot-example]

🔐 Read the blog post: https://developer.okta.com/blog/2022/08/05/spring-boot-saml[Get Started with Spring Boot and SAML]

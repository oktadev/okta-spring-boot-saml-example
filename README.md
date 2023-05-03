# Spring Boot, SAML, and Okta

A Spring Boot example app that shows how to implement single sign-on (SSO) with Spring Security's SAML and Okta.

Please read [Get Started with Spring Boot, SAML, and Okta][blog] to see how this app was created.

**Prerequisites:** 

- [SDKMAN](https://sdkman.io/) (for Java 17)

> [Okta](https://developer.okta.com/) has Authentication and User Management APIs that reduce development time with instant-on, scalable user infrastructure. Okta's intuitive API and expert support make it easy for developers to authenticate, manage and secure users and roles in any application.

* [Getting Started](#getting-started)
* [Links](#links)
* [Help](#help)
* [License](#license)

## Getting Started

To install this example application, run the following commands:

```bash
git clone https://github.com/oktadev/okta-spring-boot-saml-example.git
cd okta-spring-boot-saml-example
```

### Create a SAML App in Okta

To begin, you'll need an Okta developer account. You can create one at [developer.okta.com/signup](https://developer.okta.com/signup) or install the [Okta CLI](https://cli.okta.com) and run `okta register`.

Then, log in to your account and go to **Applications** > **Create App Integration**. Select **SAML 2.0** and click **Next**. Name your app something like `Spring Boot SAML` and click **Next**.

Use the following settings:

* Single sign on URL: `http://localhost:8080/login/saml2/sso/okta`
* Use this for Recipient URL and Destination URL: ✅ (the default)
* Audience URI: `http://localhost:8080/saml2/service-provider-metadata/okta`

Then click **Next**. Select the following options:

* I'm an Okta customer adding an internal app
* This is an internal app that we have created

Select **Finish**.

Okta will create your app, and you will be redirected to its **Sign On** tab. Scroll down to the **SAML Signing Certificates** and go to **SHA-2** > **Actions** > **View IdP Metadata**. You can right-click and copy this menu item's link or open its URL. Copy the resulting link to your clipboard. It should look something like the following:

```
https://dev-13337.okta.com/app/<random-characters>/sso/saml/metadata
```

Go to your app's **Assignment** tab and assign access to the **Everyone** group.

### Create a SAML App in Auth0

[Sign up for an Auth0 account](https://auth0.com/signup) or [log in](https://auth0.com/api/auth/login?redirectTo=dashboard) with your existing one. Navigate to **Applications** > **Create Application** > **Regular Web Applications** > **Create**.

Select the **Settings** tab and change the name to `Spring Boot SAML`. Add `http://localhost:8080/login/saml2/sso/auth0` as an **Allowed Callback URL**.

Scroll to the bottom, expand **Advanced Settings**, and go to **Endpoints**. Copy the value of the **SAML Metadata URL**. Select **Save Changes**.

Clone the `auth0` branch, which contains changes to look for Auth0 attributes instead of Okta attributes.

```shell
git clone -b auth0 https://github.com/oktadev/okta-spring-boot-saml-example.git
cd okta-spring-boot-saml-example
```

Copy your **SAML Metadata URL** into `src/main/resources/application.yml`:

```yaml
spring:
  security:
    saml2:
      relyingparty:
        registration:
          auth0:
            assertingparty:
              metadata-uri: <your-auth0-metadata-uri>
            ...
```

### Run the App and Login

Run your Spring Boot app from your IDE or using the command line:

```shell
./gradlew bootRun
```

Open `http://localhost:8080` in your favorite browser and log in with the credentials you used to create your account.

You should see a successful result in your browser.

If you want to make the logout button work and display a user's attributes, please read the blog post. 

## Links

This example uses Spring Boot and [Spring Security SAML](https://docs.spring.io/spring-security/reference/servlet/saml2/login/index.html) to integrate with Okta and Auth0. 

## Help

Please post any questions as comments on the [blog post][blog], visit the [Okta Developer Forums](https://devforum.okta.com/), or talk to us on the [Auth0 Community Forums](https://community.auth0.com/).

## License

Apache 2.0, see [LICENSE](LICENSE).

[blog]: https://developer.okta.com/blog/2022/08/05/spring-boot-saml


---
title: "OneLogin"
excerpt: "Configuration tips for OneLogin"
---
[block:api-header]
{
  "title": "Overview"
}
[/block]
This topic explains how to connect OneLogin to LaunchDarkly. 

After you complete these procedures, you can use OneLogin to manage your LaunchDarkly users, including configuring their roles and access. 
[block:api-header]
{
  "title": "Adding a LaunchDarkly SAML Application"
}
[/block]
To add a LaunchDarkly SAML application to OneLogin:

1. Log in to OneLogin.
2. Navigate to **Applications**.
3. Click the **Add App** button.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/82fb675-Screenshot_2019-08-30_OneLogin4.png",
        "Screenshot_2019-08-30 OneLogin(4).png",
        1104,
        128,
        "#9ca0a1"
      ],
      "sizing": "80",
      "caption": "The Administration screen of OneLogin."
    }
  ]
}
[/block]
3. Search for `LaunchDarkly`. The pre-configured app templates for LaunchDarkly appear.
4. Choose the `SAML 2.0, provisioning` version of the LaunchDarkly app.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/e622978-Screenshot_2019-08-30_OneLogin5.png",
        "Screenshot_2019-08-30 OneLogin(5).png",
        1087,
        292,
        "#fafbfb"
      ],
      "sizing": "80",
      "caption": "The LaunchDarkly search results."
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "SAML 2.0 is Required",
  "body": "Choose the SAML 2.0, provisioning app from the list of LaunchDarkly apps. If you choose another app, it will not work."
}
[/block]

[block:api-header]
{
  "title": "Configuring LaunchDarkly's Security Settings"
}
[/block]
To enable SAML with OneLogin, you must configure the LaunchDarkly's security settings as well as the information OneLogin has about LaunchDarkly.

To configure LaunchDarkly's security settings for SAML:

1. Open LaunchDarkly's [Security page](https://app.launchdarkly.com/settings/security).
2. Click **Edit SAML Configuration**. The **Edit your SAML configuration** screen opens.
3. Enter configuration information from OneLogin in the appropriate fields.
3. Copy the **Assertion customer service URL** to a secure place. You need it in order to configure OneLogin.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/3124156-Account_settings.png",
        "Account_settings.png",
        782,
        837,
        "#f8f8f7"
      ],
      "sizing": "80",
      "caption": "The SAML configuration screen."
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Configuring your LaunchDarkly app in OneLogin"
}
[/block]
In the SSO section of your LaunchDarkly application in OneLogin, you'll find the required values for the Sign-on URL as well as your X.509 certificate. You will need to input OneLogin's SAML 2.0 Endpoint (HTTP) value to the Sign-on URL field:

1. Log in to OneLogin.
2. Click **Applications** > **Applications**.
3. Choose the LaunchDarkly app from the list of apps.
4. Navigate to the **SSO** section of your LaunchDarkly OneLogin application.
5. Copy **OneLogin's SAML 2.0 Endpoint (HTTP)** value to a secure place.
6. Open LaunchDarkly in a separate tab. You need information from OneLogin to finish configuration.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/db8e325-Screenshot_2019-08-30_OneLogin6.png",
        "Screenshot_2019-08-30 OneLogin(6).png",
        1011,
        674,
        "#f3f5f6"
      ],
      "caption": "The SSO section of LaunchDarkly."
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Configuring the LaunchDarkly App in OneLogin"
}
[/block]
In this procedure, you will connect LaunchDarkly to OneLogin.

1. Log in to OneLogin.
2. Navigate to the **Configuration** section.
3. Enter the assertion customer service URL you saved from the previous procedure in the **Consumer URL** field.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ecfa625-Screenshot_2019-08-30_OneLogin7.png",
        "Screenshot_2019-08-30 OneLogin(7).png",
        1028,
        561,
        "#f5f6f7"
      ],
      "caption": "The OneLogin Configuration screen."
    }
  ]
}
[/block]
This connects LaunchDarkly to OneLogin.
[block:api-header]
{
  "title": "Adding Users and Setting Roles in OneLogin"
}
[/block]
Now that you have the LaunchDarkly application configured in OneLogin, all that remains is to add user access to the LaunchDarkly app. 

1. Log in to OneLogin.
2. Navigate to the **User** directory.
3. Click on the profile of the user you would like to add to LaunchDarkly.
4. Click on that user's **Applications** section.
5. Click the **+** in the top right corner.
5. Choose **LaunchDarkly** from the list. The Edit Login menu opens. 
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/d8f6662-Screenshot_2019-08-30_OneLogin8.png",
        "Screenshot_2019-08-30 OneLogin(8).png",
        1265,
        365,
        "#8f9191"
      ],
      "sizing": "80",
      "caption": "The Applications section."
    }
  ]
}
[/block]
6. (Optional) Configure the **lastName** and **firstName** fields with user information.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/aa97a6e-Screenshot_2019-08-30_OneLogin9.png",
        "Screenshot_2019-08-30 OneLogin(9).png",
        637,
        624,
        "#fafbfc"
      ],
      "sizing": "80",
      "caption": "The Edit Login screen."
    }
  ]
}
[/block]
7. Enter role information for the user. The three supported roles are:
 * `reader`
 * `writer`
 * `admin` 
8. (Optional) If you are using a custom role, enter the custom role key in the **customRole** field. If a user has multiple custom roles, add them by entering the role keys for each role, separated by commas. 
[block:callout]
{
  "type": "warning",
  "title": "Enter All of a User's Roles",
  "body": "When you are configuring a user, you must enter the complete list of the user's roles, including the roles already present in LaunchDarkly. This list overrides what is in LaunchDarkly. It does not append to an existing list in LaunchDarkly. \n\nIf you make any changes to the user's name or roles within OneLogin, they update the next time the user accesses LaunchDarkly through the OneLogin portal."
}
[/block]
9. Click **Save**. 

Now this user can access the LaunchDarkly app through OneLogin. 

If this is a new user who has never accessed LaunchDarkly before, an account will be automatically created for them when they log in through the OneLogin portal.
[block:api-header]
{
  "title": "User Provisioning with SCIM"
}
[/block]
You can also configure OneLogin to provision users with LaunchDarkly's SCIM API.
[block:callout]
{
  "type": "warning",
  "body": "You must be a LaunchDarkly administrator or account owner to complete this procedure. In addition, you must have already enabled SSO by following the procedure above.",
  "title": "Administrator Permissions and SSO are Required"
}
[/block]
1. Log in to OneLogin.
2. Access the LaunchDarkly app.
3. Click **Configuration**.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/a17f608-Screenshot_2019-08-30_OneLogin1.png",
        "Screenshot_2019-08-30 OneLogin(1).png",
        974,
        553,
        "#f5f6f7"
      ]
    }
  ]
}
[/block]
4. Click **Authenticate** in the **API Connection** section. The LaunchDarkly OAuth workflow begins.
5. Click the **LaunchDarkly (SCIM Test)** link. The LaunchDarkly OAuth authorization appears.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ccc75d4-Screenshot_2019-08-30_OneLogin2.png",
        "Screenshot_2019-08-30 OneLogin(2).png",
        974,
        638,
        "#6d6e6e"
      ],
      "caption": "The LaunchDarkly authentication process screen."
    }
  ]
}
[/block]
6. Click **Authorize** to allow OneLogin to manage your users.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/29c04f5-Screenshot_2019-08-30_Requesting_Access.png",
        "Screenshot_2019-08-30 Requesting Access.png",
        694,
        535,
        "#f7f9f9"
      ],
      "caption": "The LaunchDarkly OAuth permissions screen."
    }
  ]
}
[/block]
7. In OneLogin, navigate to the **Provisioning** tab. 
8. Confirm that the **Enable Provisioning** box is checked in the **Workflow** section.
9. Click **Save**.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ded1ed9-Screenshot_2019-08-30_OneLogin3.png",
        "Screenshot_2019-08-30 OneLogin(3).png",
        1007,
        594,
        "#e0e2e3"
      ],
      "caption": "The Provisioning screen."
    }
  ]
}
[/block]
That's it! You have successfully connected OneLogin with LaunchDarkly.
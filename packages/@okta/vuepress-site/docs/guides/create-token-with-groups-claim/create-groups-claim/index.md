---
title: Create a Groups Claim
---

You can add a groups claim for any combination of application groups and user groups into ID tokens to perform SSO using the Okta Authorization Server, or ID tokens and access tokens to perform authentication and authorization using the Custom Authorization Server (API Access Management required). This process optionally uses Okta's flexible app profile, which accepts any JSON-compliant content, to create a whitelist of groups that can then easily be referenced. This is especially useful if you have a large number of groups to whitelist or otherwise need to set group whitelists on a per-application basis.

### Create a Groups Claim for Okta-Mastered Groups
Do the following to create a Groups claim for Okta-mastered groups on an OpenID Connect client application. This approach is recommended if you are only using Okta-mastered groups.

>These steps require the administrator UI. If you are using the Developer Console, select the drop-down control on the left side of the top banner to switch to the Classic UI.

1. In the administrator UI, from the **Applications** menu, select **Applications**, and then select the OpenID Connect client application that you want to configure.

2. Navigate to the **Sign On** tab and click **Edit** in the **Open ID Connect ID Token** section.

3. In **Groups claim type**, choose either **Filter** or **Expression**.

4. In **Group claims filter**, leave the default name **groups** or change it if you want, and then add the appropriate filter or expression. For example, select **Filter**, and then select **Matches regex** and enter `.*` to return the user's groups. See [Okta Expression Language Group Functions](/docs/reference/okta-expression-language/#group-functions) for more information.

5. Add the groups claim to the scopes in your request. The ID token is returned in the response.

Request Example:
```bash
cURL -X GET \
"https://your-okta-domain/oauth2/v1/authorize?client_id=0baiz2v8m6unWCvXM0h7
&response_type=id_token
&scope=openid%20groups
&redirect_uri=http:%2F%2Flocalhost:8080
&state=myState&nonce=yourNonceValue"
```

Notes:
* You can also create a claim directly in a Custom Authorization Server instead of on the OpenID Connect or OAuth 2.0 app.
* The maximum number of groups that you can specify must be less than 100.

### Create Groups Claims with a Dynamic Whitelist

You can use the [Okta Expression Language Group Functions](/docs/reference/okta-expression-language/#group-functions) to use static and dynamic whitelists.

Three Group functions help you use dynamic group whitelists:  `contains`, `startsWith`, and `endsWith`.

These functions return all the groups that match the specified criteria. Use this function to get a list of groups that include the current user as a member.

You can use this function anywhere to get a list of groups of which the current user is a member, including both user groups and app groups that originate from sources outside Okta, such as from Active Directory and Workday. Additionally, you can use this combined, custom-formatted list for customizable claims into Access and ID Tokens that drive authorization flows. All three functions have the same parameters:

| Parameter      | Description                                                                                             | Nullable    | Example Values                                           |
| :------------- | :--------------                                                                                         | :---------- | :---------------------                                   |
| app            | Application type or App ID                                                                              | FALSE       | `"OKTA"`, `"0oa13c5hnZFqZsoS00g4"`, `"active_directory"` |
| pattern        | Search term                                                                                             | FALSE       | `"Eastern-Region"`, `"Eastern"`, `"-Region"`             |
| limit          | Maximum number of groups returned. Must be a valid EL expression and evaluate to a value from 1 to 100. | FALSE       | `1`, `50`, `100`                                         |

To use these functions to create a token using a dynamic group whitelist, create a Groups claim on an app:

1. In the administrator UI, from the **Applications** menu, select **Applications**, and then select the client application that you want to configure.

2. Navigate to the **Sign On** tab and click **Edit** in the **Open ID Connect ID Token** section.

3. In **Groups claim type**, choose **Expression**.

4. In **Group claims filter**, leave the default name **groups** or change it if you want.

5. In **Groups claim expression**, add one of the three functions with the criteria for your dynamic group whitelist:

    `Groups.startsWith("active_directory", "myGroup", 10)`

  Notes:
  * The syntax for these three functions is different from `getFilteredGroups`.
  * You can also create a claim directly in a Custom Authorization Server instead of on the OpenID Connect or OAuth 2.0 app.

### Create Groups Claims with a Static Whitelist

Before you start, perform the following two tasks for either Okta Authorization Server or Custom Authorization Server.

 * Create an OAuth 2.0 or OpenID Connect client with the [Apps API](/docs/reference/api/apps/#request-example-8). In the instruction examples, the client ID is `0oabskvc6442nkvQO0h7`.
 * Create the groups that you want to configure in the groups claim. In the instruction examples, we're configuring the `WestCoastDivision` group, and the group ID is `00gbso71miOMjxHRW0h7`.

Now, use the instructions for your chosen authorization server to create a group claim, assign a group whitelist to your client app, and configure a groups claim that references a whitelist:

* [Create a Token with a Groups Claim with Okta Authorization Server](#create-a-token-with-a-groups-claim-okta-authorization-server)
* [Create a Token with a Groups Claim with Custom Authorization Server](#create-a-token-with-a-groups-claim-custom-authorization-server)

For examples using [Okta Expression Language Group Functions](/docs/reference/okta-expression-language/#group-functions) in static whitelists, see [Use Group Functions for Static Group Whitelists](#use-group-functions-for-static-group-whitelists).

#### Use Group Functions for Static Group Whitelists

The `getFilteredGroups` group function helps you use a static group whitelist.

`getFilteredGroups` returns all groups contained in a specified list, the whitelist, of which the user is a member. The groups are returned in a format specified by the `group_expression` parameter. You must specify the maximum number of groups to return. The format of this EL function is `getFilteredGroups( whitelist, group_expression, limit)`.

You can use this function anywhere to get a list of groups of which the current user is a member, including both user groups and app groups that originate from sources outside Okta, such as from Active Directory and Workday. Additionally, you can use this combined, custom-formatted list for customizable claims into Access and ID Tokens that drive authorization flows.

This function takes Okta EL expressions for all parameters that evaluate to the correct data type. With these expressions you can create complex definitions for the whitelist, the group format, and for the number of groups to return that can include `if` logic and customized formatting.

| Parameter          | Description                                                                                                                              | Nullable |
| :----------------- | :--------------------------------------------------------------------------------------------------------------------------------------- | :------- |
| whitelist          | Valid Okta EL expression that evaluates to a string array of group ids                                                                   | FALSE    |
| group_expression   | Valid Okta EL expression that evaluates to a string to use to evaluate the group. This string must also be a valid Okta EL expression.   | FALSE    |
| limit              | Valid Okta EL expression that evaluates to an integer between 1 and 100, inclusive to indicate the maximum number of groups to return    | FALSE    |

All parameters must be valid Okta EL expressions that evaluate as described above. Okta EL expressions can be comprised of strings, integers, arrays, etc.

The string produced by the `group_expression` parameter usually contains attributes and objects from the [Groups API](/docs/reference/api/groups/), although it isn't limited to those attributes and objects. Attributes and objects listed in the [Group Attributes](/docs/reference/api/groups/#group-attributes) section of the Groups API can be any of the following: `id`, `status`, `name`, `description`, `objectClass`, and the `profile` object that contains the `groupType`, `samAccountName`, `objectSid`, `groupScope`, `windowsDomainQualifiedName`, `dn`, and `externalID` attributes for groups that come from apps such as Active Directory.

The `whitelist` parameter must evaluate to a list of group ids that is returned from the [Groups API](/docs/reference/api/groups/). If the user is not a member of a group in the whitelist, the group is ignored.

**Parameter Examples**

* whitelist
  * Array: `{"00gn335BVurvavwEEL0g3", "00gnfg5BVurvavAAEL0g3"}`<br />
  * Array variable: `app.profile.groups.whitelist`
* group_expression
  * Attribute name: `"group.id"`
  * Okta EL string containing an if condition: `"(group.objectClass[0] == 'okta:windows_security_principal') ? 'AD: ' + group.profile.windowsDomainQualifiedName : 'Okta: ' + group.name"`
      If *okta:windows_security_principal* is true for
      a group, the function returns the `windowsDomainQualifiedName` prefixed with `AD:`; otherwise, the function returns the group name prefixed with `Okta:`.
* limit
   * Integer between 1 and 100, inclusive; for example: `50`.
   * Okta EL expression containing a condition that evaluates to an integer: `app.profile.maxLimit < 100 ? app.profile.maxLimit : 100`.
    If the maximum group limit in the profile is less than 100, return that number of groups; otherwise, return a maximum of 100 groups. If there are more groups returned than the specified limit, an error is returned.

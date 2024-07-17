---
title: Configuring LDAP in Puppet
date: 2024-07-19 11:00:00 -0000
categories: [Puppet Enterprise]
tags: [Puppet, LDAP, PE]
---

# Configuring LDAP in Puppet

If you're preparing to connect your Puppet Enterprise system to your production LDAP or you just want to learn a bit more about LDAP, it's useful to test it out in a lab environment, this post is a basic walk through to get up and running with LDAP in Puppet Enterprise.

## LDAP Server

The first thing we're going to need is an LDAP server, building one from scratch will be a pain, there are Docker containers out there that will do the job, but an even simpler method is to use a free publicly available LDAP surver like [forumsys.com](https://www.forumsys.com/2022/05/10/online-ldap-test-server/) 

## Puppet Enterprise config

To configure LDAP in Puppet Enterprise, login as administrator, under **ADMIN** go to **Access control**, click the **LDAP** tab and then the blue button to **+ Add LDAP directory**. You now have a form to fill in

### LDAP Config details

#### Directory Information

| KEY                   | VALUE    |
| --------------------- | -------- |
| DIRECTORY NAME        | Forumsys |
| LOGIN HELP (OPTIONAL) | https://www.forumsys.com |

#### Connection Information

| KEY                   | VALUE    |
| --------------------- | -------- |
| HOSTNAME                     | ldap.forumsys.com |
| PORT                         | 389 |
| LOOKUP USER (OPTIONAL)       | cn=read-only-admin,dc=example,dc=com |
| LOOKUP PASSWORD              | password |
| CONNECTION TIMEOUT (SECONDS) | 10 |
| CONNECT USING                | Plain text (insecure connection) |
| BASE DISTINGUISHED NAME      | dc=example,dc=com |

#### Attribute mappings

| KEY                   | VALUE    |
| --------------------- | -------- |
| USER LOGIN ATTRIBUTE     | uid |
| USER EMAIL ADDRESS FIELD | mail |
| USER FULL NAME           | cn |

#### Querying users

| KEY                   | VALUE    |
| --------------------- | -------- |
| USER RELATIVE DISTINGUISHED NAME (OPTIONAL) | *leave blank* |

#### Querying groups

| KEY                   | VALUE    |
| --------------------- | -------- |
| GROUP OBJECT CLASS                           | groupOfUniqueNames |
| GROUP MEMBERSHIP FIELD                       | uniqueMember |
| GROUP NAME ATTRIBUTE                         | ou |
| GROUP LOOKUP ATTRIBUTE                       | cn |
| GROUP RELATIVE DISTINGUISHED NAME (OPTIONAL) | *leave blank* |
| TURN OFF LDAP_MATCHING_RULE_IN_CHAIN?        | *uncheck* |
| SEARCH NESTED GROUPS?                        | *check* |

Finally click **Save changes** and LDAP is configured!

### Login Test

Now will be a good time to test out the login, rather than completely logout of Puppet Enterprise, I find it easier to open an incognito window in chrome. Navigate to the console and try logging in.

| KEY                   | VALUE    |
| --------------------- | -------- |
| USERNAME: | tesla |
| PASSWORD: | password |

You should successfully login but be presented with the message **You are not authorized to access the console**.

### User Groups and Permissions

This will be a two step process to add an existing LDAP group mapping into Puppet Enterprise and then assign it an RBAC role.

#### Adding a group

To configure an LDAP group in Puppet Enterprise, login as administrator, under **ADMIN** go to **Access control**, click the **User groups** tab. You should see a table which already lists **forumsys** as an **Identity provider**, under **Login** type **scientists** and click **Add group**, this will only work if the LDAP configuration is correct and the scientists group exists in LDAP. 

#### Add permissions 

To configure RBAC permissions for the scientists group, login as administrator, under **ADMIN** go to **Access control**, click the **User roles** tab, click the **Viewers** role and then click the **Members groups** tab, in the drop down box select **scientists (forumsys)** and click **Add group**.

### Login Test (again)

Lets try logging in again.

| KEY                   | VALUE    |
| --------------------- | -------- |
| USERNAME: | tesla |
| PASSWORD: | password |

You should now see the Puppet Enterprise console, done!

## Troubleshooting

The main problem I had was getting the groups working, the reason for that turned out to be the group lookup attribute. How I found the correct lookup attribute was to use [Filestash's LDAP Browser tool](https://www.filestash.app/ldap-browser.html). Simply fill in the login details and you can browse all the keys and values and check they're correct.

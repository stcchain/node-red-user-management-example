# node-red-user-management-example #

This repository contains an example for user management based on Node-RED flows. While it was designed to be immediately usable with the server implemented in [node-red-within-express](https://github.com/rozek/node-red-within-express) and play well together with the authentication and authorization mechanisms described in [node-red-authorization-examples](https://github.com/rozek/node-red-authorization-examples), the example may also be used in other environments.

> Nota bene: while user ids in "node-red-within-express" and "node-red-authorization-examples" are quite flexible in their format, this example restricts user ids to email addresses!

## Prerequisites ##

The example requires the following Node-RED extensions

* [node-red-contrib-components](https://github.com/ollixx/node-red-contrib-components)<br>"Components" allow multiply needed flows to be defined once and then invoked from multiple places
* [node-red-node-email](https://github.com/node-red/node-red-nodes/tree/master/social/email)<br>allows to create and send EMails from Node-RED

Additionally, the example expects the global flow context to contain an object called `UserRegistry` which has a format similar to the one described in "node-red-within-express":

* the object's property names are the **email addresses of registered users**<br>(this specification differs from the original one!)
* the object's property values are JavaScript objects with the following properties, at least (additional properties may be added at will):
  * **Roles**<br>is either missing or contains a list of strings with the user's roles. There is no specific format for role names
  * **Salt**<br>contains a random "salt" value which is used during PBKDF2 password hash calculation
  * **Hash**<br>contains the actual PBKDF2 hash of the user's password
  * **UUID**<br>contains a [version 4 UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_(random)) which uniquely identifies a given user even after email address changes (this property has been added to the original specification)
  * **agreedToDPS**<br>indicates that the user has read and agreed to this service's "Data Privacy Statement" and must be set to `true` for an account to get confirmed (this property has been added to the original specification)
  * **agreedToTOS**<br>indicates that the user has read and agreed to this service's "Terms of Service" and must be set to `true` for an account to get confirmed (this property has been added to the original specification)

When used outside "node-red-within-express", the following flows allow such a registry to be loaded from an external JSON file called `registeredUsers.json` (or to be created if no such file exists or an existing file can not be loaded) and written back after changes:

![](outside-node-red-within-express.png)

Just import [these flows](outside-node-red-within-express.json), place them on your Node-RED workspace and - if need be - check the "Inject once" setting of the node labelled "at Startup". By default, the created user registry contains a single user `node-red@mail.de` with the password `t0pS3cr3t!` and a single role `node-red` (this is exactly the same user who is also included in "node-red-within-express" by default)

For testing and debugging purposes, the [following flow](show-user-registry.json) may also be imported, which dumps the current contents of the user registry onto Node-RED's debug console when clicked:

![](show-user-registry.png)

# Typical User Lifecycle ##

A typical user lifecycle looks as follows:

* **a new user gets registered**<br>either by him/herself or by an administrator. In this implementation, new users who register themselves only have to specify their email address
* **upon registration, an "account confirmation message" is sent** by email to the address given during registration<br>this email contains a link which, when clicked, should navigate to a web page (or web application) where the new user *may* add additional information (such as his/her real name, f.e.), *must* define a password for his/her account, *must* (read and) agree to a "Data Privacy Statement" and to some "Terms of Service" (for legal reasons) and then send that information back to the server in order to **confirm the account**. If such a confirmation does not get completed within a certain period, the registration is automatically cancelled - until then, the given email address is reserved for the given account and no other account may be registered with the same address
* after successful confirmation, **the user may "log in"** using the password defined before and use the offered service
* while logged in, a user may also
  * **change his/her email address**
  * **change his/her password**
  * **change other account details** (except their agreement to "Data Privacy Statement" and "Terms of Service")
* should a user have forgotten his/her password, he/she may **start a "password reset" process**
* last, but not least, every user may **delete his/her account**

While "normal" users may only inspect and affect their own accounts, User Administrators (i.e., users with the role `user-admin`) may also manage the accounts of other people. In particular, they may

* **register other users**
* **change other user's email addresses**
* **change other user's roles**
* **inspect or change other user's account details** and
* **delete other users**

But even User Administrators neither have access to other user's passwords nor can they change other user's agreement to the service's "Data Privacy Statement" and "Terms of Service"

> Nota bene: the legal documents mentioned above (i.e., "Data Privacy Statement" and "Terms of Service") are not part of this user management implementation but have to be provided separately

Should "Data Privacy Statement" and/or "Terms of Service" change, properties `agreedToDPS` and/or `agreedToTOS` may be set to `false` again. in that case, any new login should immediately redirect the user to a separate web document where he/she may either (read the changed documents) and agree again or delete his/her account. Without such an agreement, the user should no longer be allowed to access the service and logged out immediately.


## License ##

[MIT License](LICENSE.md)

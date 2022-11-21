# Awesome Sieve Filters

A repo of useful sieve filters by the community.
This is mainly aimed at [ProtonMail](https://mail.proton.me) filters,
but they should work with any sieve filter interpreter.
See [here](https://proton.me/support/sieve-advanced-custom-filters) for the Proton sieve filter docs.

---
## AnonDaddy specific sender
This example is using Anonaddy alias forwarding, but the field in the header can be adjusted to be SimpleLogin or even the base Proton header fields.
Placing Seen first ensures it works. If you use the expire command seen will not work unless it is first - confirmed by Proton mods.
Expire will delete email after stated days (requires vnd.proton.expire).

~~~sieve
if  header :matches "X-Anonaddy-Original-Sender" ["SENDER@EMAIL.com"]{ 
	addflag "\\Seen"; 
    fileinto "FOLDERNAME";
	expire "day" "3";
}
~~~
## File by contact group name

If the email comes from an address listed in the a contact group.
You need to create a contact group for every folder you have.

~~~sieve
if header :list "from" ":addrbook:personal?label=CONTACTGROUPNAME" 
{
  fileinto "FOLDERNAME";
  fileinto "LABEL";
}
~~~
## File by recipient

File into specific folder for a chosen recipient email address.

~~~sieve
if allof (address :all :comparator "i;unicode-casemap" :is ["To", "Cc", "Bcc"] "NAME@YOUREMAIL.COM") {
    fileinto "FOLDERNAME";
}
~~~
## File all sent emails

This prevents your sent emails showing up in other folders.
Create an addressbook called "myself" and input all your email addresses.

~~~sieve
if header :list "from" ":addrbook:myself"
{
  fileinto "sent";
  return;
}
~~~
## File unkown senders

If the email address isn't in any contact groups, move it into a folder named SCREENER

~~~sieve
if not header :list "from" ":addrbook:personal?label=*" 
{
  fileinto "Screener";
  stop;
}
~~~
## SimpleLogin Labels

This filter automatically sets label for emails sent to SimpleLogin aliases. For example, any email sent to alias `paypal.XXXX@slmail.me` will be labeled as `Paypal`.

~~~sieve
require ["include", "environment", "variables", "relational", "comparator-i;ascii-numeric", "spamtest", "fileinto"];

# Generated: Do not run this script on spam messages
if allof (environment :matches "vnd.proton.spam-threshold" "*",
spamtest :value "ge" :comparator "i;ascii-numeric" "${1}")
{
    return;
}


if allof (address :all :comparator "i;unicode-casemap" :matches "From" "*simplelogin.co",
address :all :matches "To" "*.*@*") # The "To" address condition is used to get the first part (before the dot)
{
	set :lower :upperfirst "labelvar" "${1}"; # ${1} contains the part before the dot, then we capitalize the first letter
  	fileinto "${labelvar}"; # Apply the label
}
~~~

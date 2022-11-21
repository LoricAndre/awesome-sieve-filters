# Awesome Sieve Filters

A repo of useful sieve filters by the community.
This is mainly aimed at [ProtonMail](https://mail.proton.me) filters,
but they should work with any sieve filter interpreter.
See [here](https://proton.me/support/sieve-advanced-custom-filters) for the Proton sieve filter docs.

---
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
##TBTMailSieve Additions - from twoBrokenThumbs

ProtonMail Sorting
The following are all listed as if-elseif.
Order of filters will matter.

~~~sieve
#Requirements
require ["fileinto", "extlists", "imap4flags", "vnd.proton.expire"];


#Spam - Let's Proton run its spam process before applying personalized sieve filters
if allof (environment :matches "vnd.proton.spam-threshold" "*",
spamtest :value "ge" :comparator "i;ascii-numeric" "${1}")
{
    return;
}


# Sent Email Filing (prevents your sent emails showing up in other folders)
# Create an addressbook called "myself" and input all your email addresses
if header :list "from" ":addrbook:myself"
{
  fileinto "sent";
  return;
}


# Filter emails by your inbound email address
elsif allof (address :all :comparator "i;unicode-casemap" :is ["To", "Cc", "Bcc"] "NAME@YOUREMAIL.COM") {
    fileinto "FOLDERNAME";
} 


# Filter Based on Specific From Email 
# This example is using Anonaddy alias forwarding, but the field in the header can be adjusted to be SimpleLogin or even the base Proton header fields
# Placing Seen first ensures it works. If you use the expire command seen will not work unless it is first - confirmed by Proton mods.
# expire will delete email after stated days (requires vnd.proton.expire)
elsif  header :matches "X-Anonaddy-Original-Sender" ["SENDER@EMAIL.com"]{ 
	addflag "\\Seen"; 
    fileinto "FOLDERNAME";
	expire "day" "3";
}


# If the email comes from an address listed in the a contact group
# Create a contact group for every folder you have 
elsif header :list "from" ":addrbook:personal?label=CONTACTGROUPNAME" 
{
  fileinto "FOLDERNAME";
  fileinto "LABEL";
}


# If the email address isn't in any contact groups, move it into a folder named SCREENER
elsif not header :list "from" ":addrbook:personal?label=*" 
{
  fileinto "Screener";
  stop;
}
~~~

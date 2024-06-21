# Awesome Sieve Filters

A repo of useful sieve filters by the community.
This is mainly aimed at [ProtonMail](https://mail.proton.me) filters,
but they should work with any sieve filter interpreter.
See [here](https://proton.me/support/sieve-advanced-custom-filters) for the Proton sieve filter docs.

---

<!--ts-->
* [Awesome Sieve Filters](#awesome-sieve-filters)
   * [00-ads](#00-ads)
   * [00-testing](#00-testing)
   * [01-security](#01-security)
   * [02-statements](#02-statements)
   * [AnonDaddy specific sender](#anondaddy-specific-sender)
   * [coffee](#coffee)
   * [family-and-friends](#family-and-friends)
   * [File by contact group name](#file-by-contact-group-name)
   * [File by recipient](#file-by-recipient)
   * [File all sent emails](#file-all-sent-emails)
   * [File unkown senders](#file-unkown-senders)
   * [File By Mailing List](#file-by-mailing-list)
   * [finance](#finance)
   * [food-and-drink](#food-and-drink)
   * [gaming](#gaming)
   * [government](#government)
   * [health](#health)
   * [hiking](#hiking)
   * [insurance](#insurance)
   * [investing](#investing)
   * [legal](#legal)
   * [pets](#pets)
   * [real-estate](#real-estate)
   * [reservations](#reservations)
   * [SimpleLogin Labels](#simplelogin-labels)
   * [social](#social)
   * [tech](#tech)
   * [travel](#travel)
   * [utilities](#utilities)
   * [zz-orders-and-shipping](#zz-orders-and-shipping)

<!-- Created by https://github.com/ekalinin/github-markdown-toc -->
<!-- Added by: runner, at: Fri Jun 21 14:54:02 UTC 2024 -->

<!--te-->

---
## 00-ads

 If "list-unsubscribe" header present, flag for easy manual review
 Ads is a label, NOT a folder

~~~sieve
require ["fileinto", "imap4flags"];

# allowlist - do NOT tag as advertising
if address :matches :domain "from" ["*personalcapital.com", "*robinhood.com", "*nerdwallet.com"]
{
    # do nothing
}

# If "list-unsubscribe" header present, flag for easy manual review
# Ads is a label, NOT a folder
elsif exists "list-unsubscribe"
{
    fileinto "Ads";
}

# do NOT stop executing, allow other sieves to continue processing
~~~
## 00-testing

 This sieve is useful for testing. 
 Create a contact group and name it "Self". 
 Add your personal e-mail addresses to this group.
 You can alternatively use ":addrbook:myself" 
 but I prefer to test with external addresses.

~~~sieve
require ["fileinto", "extlists", "vnd.proton.expire"];  

# This sieve is useful for testing. 
# Create a contact group and name it "Self". 
# Add your personal e-mail addresses to this group.
# You can alternatively use ":addrbook:myself" 
# but I prefer to test with external addresses.
if header :list "from" ":addrbook:personal?label=Self"   
{    
    # do things
}
~~~
## 01-security

 This sieve is high up in the execution chain, as we want to short-circuit these from other sieves and centralize all security events.
 Common subjects relevant to security events
 Commonly used security services

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# This sieve is high up in the execution chain, as we want to short-circuit these from other sieves and centralize all security events.

# Common subjects relevant to security events
if header :contains "subject" ["security alert", "security notification", "login", "sign-on", 
"sign-in", "sign in", "sign on", "email address", "email change", "password", "terms of service"] 
{
    fileinto "Security";
    expire "day" "365";
    stop;
}

# Commonly used security services
elsif address :matches :domain "from" ["*lastpass.com", "*logme.in", "*okta.com", "*accounts.google.com", 
"*1password.com", "*haveibeenpwned.com", "*nextdns.io"]
{
    fileinto "Security"; 
    expire "day" "365";
    stop;
}
~~~
## 02-statements

 If the string "statement" is found in the subject, set an expiration of 365 days.
 These are almost always informational/no-action emails and only useful for a few minutes, but set a high expiration to be safe.
 do NOT stop executing, allow other sieves to continue processing

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# If the string "statement" is found in the subject, set an expiration of 365 days.
# These are almost always informational/no-action emails and only useful for a few minutes, but set a high expiration to be safe.
if header :contains "subject" "statement"
{
    expire "day" "365";
}

# do NOT stop executing, allow other sieves to continue processing
~~~
## AnonDaddy specific sender
This example is using Anonaddy alias forwarding, but the field in the header can be adjusted to be SimpleLogin or even the base Proton header fields.
Placing Seen first ensures it works. If you use the expire command seen will not work unless it is first - confirmed by Proton mods.
Expire will delete email after stated days.

~~~sieve
require ["imap4flags", "fileinto", "vnd.proton.expire"];

if  header :matches "X-Anonaddy-Original-Sender" ["SENDER@EMAIL.com"]{ 
	addflag "\\Seen"; 
    fileinto "FOLDERNAME";
	expire "day" "3";
}
~~~
## coffee

 Commonly used coffee sites

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used coffee sites
if address :matches :domain "from" ["*counterculturecoffee.com", "*birdrockcoffee.com", "*jbccoffeeroasters.com", 
"*madcapcoffee.com", "*georgehowellcoffee.com", "*onyxcoffeelab.com", "*swiftcupcoffee.com", "*nespresso.com"]
{
    fileinto "Coffee";
    expire "day" "365";
    stop;
}
~~~
## family-and-friends

 Checks if sender is in personal address book with a "Family" group association 
 Checks if sender is in personal address book with a "Friend" group association 

~~~sieve
require ["fileinto", "extlists", "vnd.proton.expire"];  

# Checks if sender is in personal address book with a "Family" group association 
if header :list "from" ":addrbook:personal?label=Family"   
{
    # This is purely extra protection in the event the sieves don't work as expected
    if hasexpiration
    {
        unexpire;
    }    
    fileinto "Family";
    stop;
}

# Checks if sender is in personal address book with a "Friend" group association 
elsif header :list "from" ":addrbook:personal?label=Friends"   
{
    # This is purely extra protection in the event the sieves don't work as expected
    if hasexpiration
    {
        unexpire;
    }    
    fileinto "Friends";
    stop;
}
~~~
## File by contact group name

If the email comes from an address listed in the a contact group.
You need to create a contact group for every folder you have.

~~~sieve
require ["extlists", "fileinto"];

if header :list "from" ":addrbook:personal?label=CONTACTGROUPNAME" 
{
  fileinto "FOLDERNAME";
  fileinto "LABEL";
}
~~~
## File by recipient

File into specific folder for a chosen recipient email address.

~~~sieve
require ["fileinto", "comparator-i;ascii-numeric"];

if allof (address :all :comparator "i;unicode-casemap" :is ["To", "Cc", "Bcc"] "NAME@YOUREMAIL.COM") {
    fileinto "FOLDERNAME";
}
~~~
## File all sent emails

This prevents your sent emails showing up in other folders.
Create an addressbook called "myself" and input all your email addresses.

~~~sieve
require ["include", "fileinto", "extlists"];

if header :list "from" ":addrbook:myself"
{
  fileinto "sent";
  return;
}
~~~
## File unkown senders

If the email address isn't in any contact groups, move it into a folder named SCREENER

~~~sieve
require ["extlists", "fileinto"];

if not header :list "from" ":addrbook:personal?label=*" 
{
  fileinto "Screener";
  stop;
}
~~~
## File By Mailing List

If the email comes from a valid account in address group, but also is a mailing list

~~~sieve
require ["extlists", "fileinto", "vnd.proton.expire"];

If  allof (
    exists "List-Unsubscribe",
    header :list "from" ":addrbook:personal?label=GROUPNAME" 
) { 
    addflag "\\Seen";
    fileinto "FOLDERNAME";
    expire "day" "3";
}
~~~
## finance

 Delete after 30 days (no need to retain routine update emails)
 These emails get routed to the main inbox
 Commonly used financial services

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Delete after 30 days (no need to retain routine update emails)
# These emails get routed to the main inbox
if address :matches :domain "from" ["*personalcapital.com", "*nerdwallet.com", "*experian.com", "*equifax.com"]
{
    expire "day" "30";
    stop;
}

# Commonly used financial services
elsif address :matches :domain "from" ["*mtb.com", "*mtbemail.com", "*mtbalerts.com", "*mandtbank.com", "*synchronybank.com", "*synchronyfinancial.com", "*chase.com", 
"*jpmorgan.com", "*bankofamerica.com", "*marcus.com", "*visa.com", "*intuit.com", "*serve.com", "*barclaycardus.com", "*venmo.com", "*ngfcu.us", 
"*personalcapital.com", "*nerdwallet.com", "*experian.com", "*equifax.com", "*nelnet.net", "*wealthfront.com", "*paypal.com", "*citi.com", "*earnest.com"]
{
    fileinto "Finance/Banking";
    expire "day" "365";
    stop;
}
~~~
## food-and-drink

 Commonly used food vendors

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used food vendors
if address :matches :domain "from" ["*highkey.com", "*highkeysnacks.com", "*doordash.com", "*instacart.com", "*papajohns.com", 
"*yelp.com", "*grubhub.com", "*postmates.com"]
{
    fileinto "Food and Drink";
    expire "day" "365";
    stop;
}
~~~
## gaming

 Commonly used gaming services

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used gaming services
if address :matches :domain "from" ["*steampowered.com", "*xbox.com", "*nintendo.net", "*nintendo.com", "*greenmangaming.com", 
"*epicgames.com", "*ea.com", "*playstationemail.com", "*rockstargames.com", "*twitch.tv", "*ubi.com", "*blizzard.com", "*bethesda.net"]
{
    fileinto "Gaming";
    expire "day" "365";
    stop; 
}
~~~
## government

 Don't expire any emails here.
 Handle the national parks service
 Technically not _exclusively_ a government domain, but most US states use them
 This should catch all US government variations

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Don't expire any emails here.

# Handle the national parks service
if address :matches :domain "from" ["*recreation.gov"]
{
    fileinto "Hiking";
    stop; 
}

# Technically not _exclusively_ a government domain, but most US states use them
if address :matches "from" ["*@*.us"]
{
    # This is purely extra protection in the event the sieves don't work as expected
    if hasexpiration
    {
        unexpire;
    }
    fileinto "Government";
    stop;
}

# This should catch all US government variations
elsif address :matches "from" ["*@*.gov"]
{
    # This is purely extra protection in the event the sieves don't work as expected
    if hasexpiration
    {
        unexpire;
    }
    fileinto "Government";
    stop;
}
~~~
## health

 Commonly used health services

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used health services
if address :matches :domain "from" ["*carefirst.com", "*ibx.com", "*ibx2.com", "*metlife.com", "*eyemed.com", "*vsp.com", "*teladoc.com", "*lenscrafters.com"]
{
    fileinto "Health";
    expire "day" "365";
    stop; 
}
~~~
## hiking

 Commonly used hiking services

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used hiking services
if address :matches :domain "from" ["*garmin.com", "*gaiagps.com", "*nextmilemeals.com", "*alltrails.com", "*opensummit.com"]
{
    fileinto "Hiking";
    expire "day" "365";
    stop;
}
~~~
## insurance

 Commonly used insurance providers

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used insurance providers
if address :matches :domain "from" ["*allstate.com", "*allstate-email.com", "*geico.com", "*geicomail.com", "*jminsure.com", 
"*travelguard.com", "*nationwide.com", "*progressive.com", "*statefarm.com", "*travelers.com", "*assurant.com"]
{
    fileinto "Finance/Insurance";
    expire "day" "365";
    stop;
}
~~~
## investing

 For Robinhood Snacks
 Route to main inbox
 Commonly used investing platforms

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# For Robinhood Snacks
# Route to main inbox
if header :contains "from" "Robinhood Snacks"
{
    expire "day" "3";
    stop;
}

# Commonly used investing platforms
elsif address :matches :domain "from" ["*troweprice.com", "*e-vanguard.com", "*vanguard.com", "*m1finance.com", "*coinbase.com", "*robinhood.com", "*fidelity.com", "*prudential.com"]
{
    fileinto "Finance/Investing";
    expire "day" "365";
    stop; 
}
~~~
## legal

 Commonly used legal platforms

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used legal platforms
if address :matches :domain "from" ["*docusign.net"]
{
    # This is purely extra protection in the event the sieves don't work as expected
    if hasexpiration
    {
        unexpire;
    }
    fileinto "Legal";
    stop;
}
~~~
## pets

 Commonly used pet vendors

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used pet vendors
if address :matches :domain "from" ["*petco.com", "*litterbox.com", "*chewy.com", "*litter-robot.com", "*foundanimals.org", 
"*openfarmpet.com", "*truthaboutpetfood.com", "*found.org"]
{
    fileinto "Pets";
    expire "day" "365";
    stop;
}
~~~
## real-estate

 Commonly used real estate platforms

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used real estate platforms
if address :matches :domain "from" ["*zillow.com", "*better.com", "*redfin.com"]
{
    fileinto "Real Estate";
    expire "day" "365";
    stop; 
}
~~~
## reservations

 General catch all for appointments
 More targeted, specific reservation websites

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# General catch all for appointments
if header :contains "subject" ["appointment"] 
{
    fileinto "Reservations";
    expire "day" "365";
    stop; 
}

# More targeted, specific reservation websites
elsif address :matches :domain "from" ["*seatme.com", "*opentable.com", "*resy.com", "*exploretock.com"]
{
    fileinto "Reservations";
    expire "day" "365";
    stop; 
}
~~~
## SimpleLogin Labels

This filter automatically sets label for emails sent to SimpleLogin aliases. For example, any email sent to alias `paypal.XXXX@slmail.me` will be labeled as `Paypal`.

~~~sieve
require ["include", "environment", "variables", "relational", "comparator-i;ascii-numeric", "spamtest", "fileinto"];

if allof (address :all :comparator "i;unicode-casemap" :matches "From" "*simplelogin.co",
address :all :matches "To" "*.*@*") # The "To" address condition is used to get the first part (before the dot)
{
	set :lower :upperfirst "labelvar" "${1}"; # ${1} contains the part before the dot, then we capitalize the first letter
  	fileinto "${labelvar}"; # Apply the label
}
~~~
## social

 Commonly used social networks

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used social networks
if address :matches :domain "from" ["*linkedin.com", "*facebookmail.com", "*facebook.com", "*reddit.com", "*twitter.com", "*instagram.com", "*pinterest.com", "*meetup.com", "*tumblr.com", "*ycombinator.com"]
{
    fileinto "Social";
    expire "day" "365";
    stop;
}
~~~
## tech

 Commonly used tech services
 Security has it's own folder - don't put tech security services here.

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used tech services
# Security has it's own folder - don't put tech security services here.
if address :matches :domain "from" ["*digitalocean.com", "*github.com", 
"*cloudflare.com", "*docker.com", "*heroku.com", "*keybase.io", "*adobe.com", 
"*netflix.com", "*google.com", "*youtube.com", "*apple.com", "*roku.com", 
"*namecheap.com", "*microsoft.com", "*ebay.com", "*spotify.com", "*plex.tv", "*logitech.com"]
{
    fileinto "Tech";
    expire "day" "365";
    stop;
}
~~~
## travel

 Commonly used travel services

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used travel services
if address :matches :domain "from" ["*southwest.com", "*hyatt.com", "*airbnb.com", 
"*lyftmail.com", "*uber.com", "*hertz.com", "*alaskaair.com", "*delta.com", 
"*goalamo.com", "*avis.com", "*ihg.com", "*kimptonhotels.com", "*hotels.com", 
"*marriott.com", "*tripadvisor.com", "*aa.com", "*autoslash.com", "*priceline.com"]
{
    fileinto "Travel";
    expire "day" "730";
    stop; 
}
~~~
## utilities

 Commonly used utilities

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# Commonly used utilities
if address :matches :domain "from" ["*bge.com", "*comcast.net", "*xfinity.com", "*t-mobile.com", "*att.com", "*spectrum.com", "*verizon.com", "*verizonwireless.com", "*rentcafe.com", "*duke-energy.com"]
{
    fileinto "Finance/Utilities";
    expire "day" "365";
    stop;
}
~~~
## zz-orders-and-shipping

 General catch all for delivery events. 
 Delete after 180 days, as this info can almost always be retrieved from the source of truth
 Sieve ordering is important here, as you can receive an email that contains "order delivered" in the subject (which would conflict with the "order handling" sieve that follows).
 General catch all for orders.
 Do not expire, as these emails might be historically significant.
 More targeted for catching general communication and updates. 
 Delete after 180 days, as this info can almost always be retrieved from the source of truth.

~~~sieve
require ["fileinto", "imap4flags", "vnd.proton.expire"];

# General catch all for delivery events. 
# Delete after 180 days, as this info can almost always be retrieved from the source of truth
# Sieve ordering is important here, as you can receive an email that contains "order delivered" in the subject (which would conflict with the "order handling" sieve that follows).
if header :contains "subject" ["shipping", "shipped", "delivered", "delivery"]
{
    # folder
    fileinto "Orders and Shipping";
    # label
    fileinto "Deliveries"; 
    expire "day" "180";
    stop;
}

# General catch all for orders.
# Do not expire, as these emails might be historically significant.
if header :contains "subject" ["order", "invoice", "receipt"]
{
    # This is purely extra protection in the event the sieves don't work as expected
    if hasexpiration
    {
        unexpire;
    }
    # folder
    fileinto "Orders and Shipping";
    # label
    fileinto "Orders";
    stop;
}

# More targeted for catching general communication and updates. 
# Delete after 180 days, as this info can almost always be retrieved from the source of truth.
elsif address :matches :domain "from" ["*ups.com", "*upsemail.com", "*usps.com", "*fedex.com", "*narvar.com", "*etsy.com", "*amazon.com", "*newegg.com", "*rei.com", "*target.com", "*prana.com", "*walmart.com"]
{
    fileinto "Orders and Shipping"; 
    expire "day" "180";
}
~~~
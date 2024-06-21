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

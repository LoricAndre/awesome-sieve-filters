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

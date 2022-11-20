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
address :all :matches "To" "*.*@*")
{
	set :lower :upperfirst "labelvar" "${1}";
  	fileinto "${labelvar}";
}
~~~ sieve

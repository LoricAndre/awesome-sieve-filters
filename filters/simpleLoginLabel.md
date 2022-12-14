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

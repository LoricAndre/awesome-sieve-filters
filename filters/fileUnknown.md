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

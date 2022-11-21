## File unkown senders

If the email address isn't in any contact groups, move it into a folder named SCREENER

~~~sieve
if not header :list "from" ":addrbook:personal?label=*" 
{
  fileinto "Screener";
  stop;
}
~~~

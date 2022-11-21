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

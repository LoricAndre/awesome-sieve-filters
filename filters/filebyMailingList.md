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

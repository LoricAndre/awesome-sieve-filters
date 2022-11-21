## File all sent emails

This prevents your sent emails showing up in other folders.
Create an addressbook called "myself" and input all your email addresses.

~~~sieve
if header :list "from" ":addrbook:myself"
{
  fileinto "sent";
  return;
}
~~~

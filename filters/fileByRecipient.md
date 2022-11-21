## File by recipient

File into specific folder for a chosen recipient email address.

~~~sieve
require ["fileinto", "comparator-i;ascii-numeric"];

if allof (address :all :comparator "i;unicode-casemap" :is ["To", "Cc", "Bcc"] "NAME@YOUREMAIL.COM") {
    fileinto "FOLDERNAME";
}
~~~

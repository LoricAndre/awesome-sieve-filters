Awesome-sieve-filters
===

<!--toc:start-->
- [Awesome-sieve-filters](#awesome-sieve-filters)
  - [Contributing](#contributing)
  - [Guidelines](#guidelines)
  - [Roadmap](#roadmap)
<!--toc:end-->

---

A repo of useful sieve filters by the community.
This is mainly aimed at [ProtonMail](https://mail.proton.me) filters,
but they should work with any sieve filter interpreter.
See [here](https://proton.me/support/sieve-advanced-custom-filters) for the Proton sieve filter docs.

## Contributing

To contribute your sieve filter, open a pull request. In the `filters` folder,
add your filter as a markdown file with the following template :

```markdown
### Filter name

Description

~~~sieve
Commented (if necessary) filter
~~~
```

## Guidelines

Please respect these rules :

- Try to split your filters as much as possible to make importing and using them as easy as possible (avoid contributing filters with multiple uses in the same file)
- Try to mark clearly in your snippets where the user needs to replace with his information (for example, email addresses)
- Comment your filters where necessary

## Roadmap

- [ ] Add more filters
- [ ] Organize the filters into categories
- [ ] Create a pull request template
- [ ] Create an issue for filter ideas or requests
- [ ] Add GitHub action for publishing
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

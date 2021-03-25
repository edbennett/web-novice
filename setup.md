---
title: Setup
---

To follow this lesson, you will need to have a Unix shell and Python installed.
If you do not already have these set up, then please follow the instructions at
[the Carpentries workshop template][workshop-template]

## Requests and BeautifulSoup4

We will be using the `requests` and `beautifulsoup4`
libraries. Please install them with your Python package manager;
for example, with `pip`, use: 

~~~
$ pip install --user requests beautifulsoup4
~~~
{: .language-bash}

or with Anaconda use:

~~~
$ conda install requests beautifulsoup4
~~~
{: .language-bash}


## Met Office API Keys

We will be making requests from the Met Office Data point.
In order to follow this section you will need a valid "API key"
for this data source. To obtain this:

- Sign up with the [Met Office Data Point][datapoint]
  You will have to choose username and password,
  and will receive an email to activate your account.
- Once your account is activated,
  navigate to the [My Account Page][metaccount].
  At the bottom of the page,
  you will find your Application Key.
  Save that API key in a plain text file,
  making sure not to share its contents with anyone.
  In the following, we will refer to this file 
  with `metoffice-api-key.txt`.


## GitHub Personal Access Token

We will be connecting to GitHub's API in this lesson. To do this, you will need
an Access Token. To get one:

- Visit [GitHub][github] and log in.
- In Settings > Developer Settings > Personal Access Token,
  click on "Generate new token". 
- In the options, tick only 
  the "public_repo" option under "repo", 
  write a sensible note,
  and click on "Generate Token"
  at the bottom of the page.
- Save it into a file,
  as you will not be able to see it again.
  In the following, we will refer to this file 
  with `github-access-token.txt`.
  Remember to keep this file safe,
  or to delete it after the lesson, as anyone with access to the token can use
  it to manage your account and repositories.
  You can also delete it from the 
  Settings > Developer Settings > Personal Access Token
  page on GitHub, to be 100% sure.
  


{% include links.md %}

[datapoint]: https://www.metoffice.gov.uk/services/data/datapoint
[github]: https://github.com
[metaccount]: https://register.metoffice.gov.uk/MyAccountClient/account/view
[workshop-template]: https://carpentries.github.io/workshop-template

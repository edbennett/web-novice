---
title: Setup
---

To follow this lesson, you will need to have a Unix shell and Python installed.
If you do not already have these set up, then please follow the instructions at
[the Carpentries workshop template][workshop-template]

### Requests and BeautifulSoup4

The `requests` and the `beautifulsoup4`
libraries are required.
Please install them with your Python package manager,
e.g., via pip, with 
~~~
pip install --user requests beautifulsoup4
~~~
{: .language-bash}
or with Anaconda.


### Met Office API Keys

In order to be able to follow
along in the episode on requests,
it is necessary 
that you obtain a valid api key
from the Metoffice Data Point.
- Sign up with the [Metoffice Data Point](https://www.metoffice.gov.uk/services/data/datapoint).  
  You will have to choose username and password,
  and will receive an email to activate your account.
- Once your account is activated,
  navigate to the [My Account Page](https://register.metoffice.gov.uk/MyAccountClient/account/view).  
  At the bottom of the page,
  you will find your Application Key.
  Save that API key in a plain text file,
  making sure not to share its contents with anyone.
  In the following, we will refer to this file 
  with `metoffice-api-key.txt`.


### GitHub Personal Access Token
- Visit [github](www.github.com) and log in
- In Settings > Developer Settings > Personal Access Token,
  click on "Generate new token". 
- In the options, tick only 
  the "public_repo" option under "repo", 
  write a sensible note 
  and click on "Generate Token"
  at the bottom of the page.
- Save it into a file,
  as you will not be able to see it again.
  In the following, we will refer to this file 
  with `github-access-token.txt`.
  Remember to keep this file safe,
  or to delete it after the lesson.
  You can also delete it from the 
  Settings > Developer Settings > Personal Access Token
  page on github, to be 100% sure.
  


{% include links.md %}
[workshop-template]: https://carpentries.github.io/workshop-template

Process for setting up github pages with namecheap domain.

1. Go to namecheap.com, select and buy domain name. 
2. Login to namecheap, go to username drop down and select dashboard.
3. Go to DomainList
4. Click manage button
5. Click Advanced DNS tab
6. Click add record and add three records: 
	Type: A Record | Host: @ | Value: 192.30.252.153 | TTL: Automatic

	Type: A Record | Host: @ | Value: 192.30.252.154 | TTL: Automatic

	Type: CNAME Record | Host: www | Value: username.github.io. | TTL: Automatic

	NOTE: CNAME record value must have a '.' at the end of it.
7. Click Save changes. 
8. Go to github project (or make new one) and add file called CNAME. In it add the domain name you just purchased. I found that if I didn't add "www" in the beginning then I could access it through "username.github.io/projectName" which would redirect to the namecheap domain but not directly through the namecheap domain. So remember to add "www" in the beginning of the CNAME file!
8. Add an index.html to your projects root.
9. Go to project settings>Github Pages>Source and change it from none to master branch. Save. 
9. Enjoy!


Helpful resources: 
https://gist.github.com/mapsam/ce60b87eea561ea6bdbf
https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages
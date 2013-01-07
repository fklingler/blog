---
layout: post
title: Deploy a Jekyll site on your own server easily
tags: jekyll, git, deployment, server
draft: true
---

When I looked at [Jekyll](http://www.jekyll.rb) to generate my static blog, the obvious choice to host it was to deploy it on the [GitHub Pages](http://pages.github.com). But for many reasons (including to keep my domain pointing to my server), I didn't want to do it that way, and wanted to host it on my own server.

I then looked at an easy way to deploy my blog[^1], if possible just by doing a `git push`, just like you'd do it if you hosted the site on GitHub Pages.

To do so, you must first create a bare git repository on the server, on which you will be able to push (if the repository is not bare, you are not able to push to it).

{: .terminal}
~~~
server:~/www/$ git init --bare blog.git
~~~

Since it is a bare repository, it will not have any index and you will not have any `_site` directory available to serve with your web server.  
To be able to access to the `_site` directory on the server, you can just clone the repository after each push. You can do that easily with a `post-receive` git hook on the server repository :

{: .terminal}
~~~
server:~/www/$ vim blog.git/hooks/post-receive
~~~

{% highlight sh %}
#!/bin/sh
GIT_REPO=$HOME/www/blog.git
TMP_GIT_CLONE=$GIT_REPO/tmp_clone
PUBLIC_WWW=$HOME/www/blog

git clone $GIT_REPO $TMP_GIT_CLONE
rsync -av --delete-after $TMP_GIT_CLONE/_site/ $PUBLIC_WWW/
rm -rf $TMP_GIT_CLONE
{% endhighlight %}

This script is quite simple, it just clones the repository to a temp directory, copy the `_site` directory to the directory served by the http server (here `~/www/blog`), and finally clean the temp directory.\\
`rsync` is used here to have the lowest downtime possible on the site, compared to delete the previous directory then move the new one.

Now, just by doing a push to the server, you site will be automatically updated!

{: .terminal}
~~~
pc:~/blog$ git remote add origin ssh://server/~/www/blog.git
pc:~/blog$ git push -u origin master
~~~

There is one problem here: you must commit the changes on the `_site` directory, so you must run `jekyll` before committing your changes.\\
It differs from the way you would do it with GitHub Pages, where you can put the `_site` directory in the `.gitignore` file, since GitHub Pages will process your files with `jekyll` on its own.\\
I don't think it's a huge problem, since I run `jekyll --auto --server` when I write my posts to be able to look how my posts render, but if you really want it you can write a pre-commit hook to run `jekyll` and add the `_site` directory to the staging index before you commit.  
Be careful, it can give you some trouble when you are not in a classic writing post workflow!

{: .terminal}
~~~
pc:~/blog$ vim .git/hooks/pre-commit
~~~

{% highlight sh %}
#!/bin/sh
jekyll --no-auto
git add _site
{% endhighlight %}

Here we are, you can now just write a post, commit them, push, and your post will be on the web!

---

[^1]: Note that you can use this method with any site you have, would you use Jekyll or not.
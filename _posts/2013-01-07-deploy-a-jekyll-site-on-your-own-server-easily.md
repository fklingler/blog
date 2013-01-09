---
layout: post
title: Deploy a Jekyll site on your own server easily
tags: jekyll, git, deployment, server
draft: true
---

When I thought about [Jekyll](http://www.jekyll.rb) to generate my static blog, the obvious choice to host it was to deploy it on the [GitHub Pages](http://pages.github.com). But for many reasons (including to keep my domain pointing to my server), I didn't want to do it that way and wanted to host it on my own server.

I then looked for an easy way to deploy my blog[^1], if possible just by doing a `git push`, just like you'd do it if you hosted the site on GitHub Pages.

To do so, you must create a **bare** git repository on the server first, to which you will be able to push your site (if the repository is not bare, you are not able to push to it).

{: .terminal}
~~~
server:~/www/$ git init --bare blog.git
~~~

Since it is a bare repository, it won't have any index and you won't have any `_site` directory available to be served by your web server.  
To be able to access to the `_site` directory on the server, you can just clone the repository after each push. You can easily do that with a `post-receive` git hook on the server repository :

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

This script is quite simple, it just clones the repository to a temp directory, copies the `_site` directory to the directory served by the http server (here `~/www/blog`), and finally deletes the temp directory.\\
`rsync` is used here to have the shortest possible downtime on the site, compared to deleting the previous directory then moving the new one.

Now, just by pushing to the server, you site will be automatically updated!

{: .terminal}
~~~
pc:~/blog$ git remote add origin ssh://server/~/www/blog.git
pc:~/blog$ git push -u origin master
~~~

But there is one problem here: you must commit the changes in the `_site` directory, so you must run `jekyll` before committing your changes.\\
It differs from the way you would do it with GitHub Pages, where you can put the `_site` directory in the `.gitignore` file, since GitHub Pages will process your files with `jekyll` on its own.\\
It's a huge problem for me, since I run `jekyll --auto --server` when I write my posts to check how they are rendered, but if you really want it you can write a pre-commit hook to run `jekyll` and add the `_site` directory to the staging index before you commit.  
Be careful, it can cause you some trouble when you are not in a classic writing post workflow!

{: .terminal}
~~~
pc:~/blog$ vim .git/hooks/pre-commit
~~~

{% highlight sh %}
#!/bin/sh
jekyll --no-auto
git add _site
{% endhighlight %}

Here we are, you can now just write a post, commit it, push, and your post will be on the web!

---

[^1]: Note that you can use this method with any of your sites, would you use Jekyll or not.

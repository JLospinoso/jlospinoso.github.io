---
layout: post
title: Jekyll via Github
date: 2015-03-01 15:00
image: /images/jekyll.png
tag: Github will host your Jekyll blog for free!
categories: [developing, blogging, github, jekyll]
---
[1]: http://jekyllrb.com/
[2]: https://help.github.com/articles/using-jekyll-with-pages/
[3]: https://github.com/jekyll/jekyll/wiki/sites
[4]: https://github.com/JLospinoso/jlospinoso.github.io
[5]: http://import.jekyllrb.com/docs/home/
[6]: http://daringfireball.net/projects/markdown/syntax
[7]: https://wordpress.com/
[8]: https://help.github.com/articles/using-jekyll-with-pages/#installing-jekyll
[9]: https://www.ruby-lang.org/
[10]: http://bundler.io/
[11]: https://github.com/join
[12]: https://pages.github.com/

### What is Jekyll
[Jekyll][1] is an easy to use, "blog-aware static site generator." What this means practically is that once you set up your blog the way you like it, submitting new blog posts is as easy as writing a plain-text document in [Markdown][6] and popping it into a `_posts` folder. Jekyll will parse all of the plain-text files and generate your blog, ready for publication. Writing a new entry is as easy as this:

	---
	layout: post
	title: My first Jekyll post
	date: 2015-03-01 15:00
	categories: [blogging, jekyll]
	---
	# My first Jekyll post
	It's pretty easy to use Markdown:
	
	* Once you get used to the syntax,
	* it's a breeze to make lists!
	
	And also to hyperlink to my favorite sites, like <http://lospi.net>.

# Why bother?
It's not for everyone, but there are some serious advantages to the Jekyll way:

* You get to build your post in Markdown, meaning you have fine-grained control over how your page gets built without having to deal with clunky markup (like HTML). If you're used to Github-flavored Markdown already, it should be fairly clear how powerful this feature is.
* Your site is a full-blown static site, meaning you can CSS style to your heart's content, add whizz-bang JS, and bump `<span>`s around all you'd like. Of course, you can just use one of the many, many available [examples][2].
* You can put your blog under source control. This makes it really, really easy to keep drafts and to backup your site. As we'll see with Github Pages hosting, publishing your updates is as easy as `git push`.
* You own every byte of your blog--rather than worrying about whether [Wordpress][7] will ever renege on their export API.

I've been experimenting with Jekyll for a few days now and [decided to make the switch][4].

# Configuring a Local Instance
It is imperative to get a local instance of Jekyll running. This will allow you to iterate between your plain-text blog and the rendered pages without having to push your changes remotely. You can follow [these instructions][8] to get it set up. Basically, you need three things:

1. [Ruby][9] installed. I tried a few versions before I finally got my setup working. I'm running Windows 7 x64 and Ruby 2.0.0 x86 ended up doing the trick (I tried more up to date versions, x64 versions, etc. until settling on this particular flavor).
2. [Bundler][10], which makes installations of Ruby *gems* much simpler. If you're wondering what didn't work with various versions of Ruby (see #1 above), it was Bundler. Once your path is setup, installing Bundler should be as easy as `gem install bundler`.
3. [Jekyll][1], which can be automagically installed with a correctly configured *Gemfile*. We'll just work off a template, so hold off on this one.

# Working off a template
You can browse the many, many available examples on [Jekyll Sites][3], or work off the most current version of [lospi.net][4]. *Note: you must sign up for a [Github account][11].*

Navigate to the parent directory you'd like to check the blog out into, and checkout one of the blogs, e.g. with the command:

	git clone git@github.com:JLospinoso/jlospinoso.github.io.git

You can now navigate into the site's folder, e.g. `jlospinoso.github.io.git`, and start a local webserver:

	cd jlospinoso.github.io.git
	bundle exec jekyll serve

Fire up your favorite web browser and navigate to `http://127.0.0.1:4000/`.

You can inspect the contents of the cloned repository to get an idea of how Jekyll generates content. The *finished product* is contained in the `_sites` folder (and should not be manually edited). Check out [Jekyll][1] documentation for an exhaustive list of the powerful things you can do--but here are a few items to get you started:

* Try creating a new blog post. Navigate to the `_posts` directory and create a new file (following the date-title format of the other posts). Use the template shown earlier in this post, copy the format of one of the other files in `_posts`, or experiment on your own!
* Look through `_layouts` to get an idea of how Jekyll converts the posts into fully marked-up HTML. Edit the `default.html` so that the site matches your name, site URL, etc.
* Edit the `css/screen.css` file to give the site a new look.
	
# Migrating from another sites
If you have a blog hosted on one of the common sites, [JekyllImport][5] can be used to turn their export dump into Markdown files that you can insert directly into `_posts`. I did this for Wordpress (.com), and the process was fairly painless. It will take a bit of manual manipulation to reverse the HTML back into Markdown if you want to make the posts match the new ones, though.

# Hosting with Github
Now that you've got a local Jekyll-generated site, you'll want to make it available to the public. It's easy and free to do this with [Github Pages][12]. You've already signed up for an account earlier, so just follow the instructions [here][12]. 

Login to Github and create your repository with `username.github.io` where `username` is, you guessed it, your username. Pull down your repository:

	git clone https://github.com/username/username.github.io

You can now transfer all the files from the template you cloned earlier into your personalized *github.io* project. If you're not too familiar with *git*, this should get you started:
	
	git add --all
	git commit -m "initial"
	git push -u origin master

Navigate a browser to `https://username.github.io/` and marvel at how easy it is to fire off a new blog post!

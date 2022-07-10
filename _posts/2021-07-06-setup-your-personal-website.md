---
layout: post
title: Setup Your Personal Website For Free
subtitle: How to use Jekyll and GitHub Pages
tags: [howto, dev]
---


In this post I will show you how I created my first personal website on GitHub Pages using Jekyll.

## What is GitHub Pages and Jekyll?

#### GitHub Pages
[GitHub](https://github.com/) is a Git (a version control system) repository hosting service that provides a Web-based graphical interface.

[GitHub Pages](https://pages.github.com/) are public webpages hosted for free through GitHub.
It takes HTML, CSS, and JavaScript files directly from a repository on GitHub, and publishes a website.

#### Jekyll
[Jekyll](https://github.com/jekyll/jekyll) is a static site generator, a.k.a, a tool that generates a full HTML static website ready to be served, based on your content and a set of templates.
Jekyll comes with built-in support for GitHub Pages, which makes it perfect to setup a personal GitHub Page.


## Requirements
This guide assumes you know a little about version control (Git and GitHub) and the basics of HTML and CSS, since we'll be working directly with these technologies.
It is also helpful to know a bit of [Markdown](https://www.markdownguide.org/), but if not, you will learn by doing.

Before we start, create an account on GitHub!


## Getting started with GitHub Pages
#### Step 1. Create a new repository
In the repository name field use your github username as such: `your-username.github.io`

Use the settings:
✅ Public
✅ Add a README file

#### Step 2. Make sure your site is up and running
Go into your new repository and go to **Settings | Pages** and check if you see the following:
> Your site is published at `<your-username>.github.io`

*(this may take a while)*

#### Step 3. Make some changes
Open the `README.md` file that was automatically created, and add to the file:

```markdown
# The beginning of my website
I can't wait to see the final product!
```

#### Step 4. Save changes
Commit `README.md`.

At the bottom of the page, there is a text input area to add a description of your changes and a button to `commit` the file.
Add a description like "My first commit" and click `Commit`.

This should be done every time we want to save any changes.

#### Step 5. Visit your website
In your browser go to `<your-username>.github.io`, do you see what you wrote in the `README.md`?

Great, you have a website up and running!

## Getting started with Jekyll
In order for Jekyll to work with our site, we need to follow [Jekyll's directory structure](http://jekyllrb.com/docs/structure/).

#### Step 6. First, create a `.gitignore` file
This file tells Git to ignore the `_site` directory created automatically by Jekyll.
Create a `.gitignore` file at the root of your project and add to the file:

```markdown
_site/
```

#### Step 7. Create a `_config.yml` file that tells Jekyll some basics about your project
Create a `_config.yml` file at the root of your project and add to the file:

```yaml
name: your-name
markdown: kramdown

# Your contacts
author: your-name
email: your-username@gmail.com
linkedin: https://www.linkedin.com/in/your-username/
github: https://github.com/your-username
```
Make sure you replace `your-name` and `your-username` for your respective name and your email, LinkedIn and GitHub accounts username.

#### Step 8. Create a `_layouts` directory, and create a file inside it called `default.html`

To add a new folder and file in the same command on GitHub, just click **Add file | Create new file** and in the `Name your file` textbox add the name of the folder followed by `/` and the name of the file.
Like this: `_layouts/default.html`

This `default.html` will be our main **layout**.

Layouts are templates that can be used by any page in our site and wrap around your content.
We will see more about this in a bit.

In `default.html` add:

{% raw %}
```html
<!DOCTYPE html>
<html>
	<head>
   		<title>{{ page.title }}</title>
 	</head>

 	<body>
 		<nav>
	    	<ul>
 	        	<li><a href="/">Home</a></li>
 	        	<li><a href="/posts">Posts</a></li>
 	        	<li><a href="/work">Work</a></li>
 	    	</ul>
 		</nav>

   	<!-- container with page content -->
 		<div class="container">

   			{{ content }}

 		</div>

 		<footer>
 	    	<ul>
 	        	<li><a href="mailto:{{ site.email }}">Email</a></li>
         		<li><a href="{{ site.linkedin }}">Linkedin</a></li>
         		<li><a href="{{ site.github }}">Github</a></li>
 			</ul>
 		</footer>
 	</body>
</html>
```
{% endraw %}

Take note of the {% raw %}`{{ page.title }}`{% endraw %} and {% raw %}`{{ content }}`{% endraw %} tags in there.
They're what Jekyll calls *Liquid* tags, and these are used to inject content into the final web page.

#### Step 9. Add an `index.md`
This will be the first page on your website, and it is calling the `default.html` layout we created in the previous step.
Add an `index.md` at the root of your project and add to the file:

{% raw %}
```markdown
---
layout: default
title: Home
---
# Hi there, welcome to my page!

See my posts at my <a href="/blog">blog</a> and read about my <a href="/work">work</a>.

```
{% endraw %}

#### Step 10. Go to your website
Go again to your website and see all these changes.

*Note: it sometimes takes a few minutes for the changes to go through. If you don’t see your changes immediately, wait a few minutes and try again.*

If you click on **blog** or **work** hyperlink it will throw a 404 code, telling you it cannot find what you are asking for.
This is because we haven't created the `blog` page nor the `work` page! Let's get into it next.

Before, let's add a bit of style to our content.


#### Step 11. Styling our content
To style our content let's create a new file named `css/main.css` at the root of the project.
Add to that file:

```css
body {
    margin: 60px auto;
    width: 70%;
}
nav ul, footer ul {
    font-family:'Helvetica', 'Arial', 'Sans-Serif';
    padding: 0px;
    list-style: none;
    font-weight: bold;
}
nav ul li, footer ul li {
    display: inline;
    margin-right: 20px;
}
a {
    text-decoration: none;
    color: #999;
}
a:hover {
    text-decoration: underline;
}
h1 {
    font-size: 3em;
    font-family:'Helvetica', 'Arial', 'Sans-Serif';
}
p {
    font-size: 1.5em;
    line-height: 1.4em;
    color: #333;
}
footer {
    border-top: 1px solid #d5d5d5;
    font-size: .8em;
}

ul.posts {
    margin: 20px auto 40px;
    font-size: 1.5em;
}

ul.posts li {
    list-style: none;
}
```

To tell our website we want to use this `.css` to style the content, we have to add the link to it in `default.html` we created earlier.

Open `default.html` and between the `<head>` and `<\head>` tags add the line:
```html
<link rel="stylesheet" type="text/css" href="/css/main.css">
```

It should look like:

```html
<!DOCTYPE html>
   <html>
   	<head>
   		<title>{{ page.title }}</title>
   		<!-- link to main stylesheet -->
   		<link rel="stylesheet" type="text/css" href="/css/main.css">
   	</head>

...
```

Don't forget to commit the changes. Always!

✅ Check again your website. Much better right!?



## Creating other pages
#### Step 12. Create a **Work** page
Similar to the `index.md` we created in [Step 9.](#step-9-finally-add-an-indexmd-at-the-root-of-your-project), we can create other pages.
Let's create a **Work** page by adding a `work.md` to the root of your project.


{% raw %}
```markdown
---
layout: default
title: Work
permalink: /work/index.html
---
## Hello

You can see my professional experience on <a href="{{ site.linkedin }}">LinkedIn</a>.

```
{% endraw %}
Save the changes and go to your website to see the results!



#### Step 13. Create a **Blog** page
Let's create the **Blog** page by adding a `blog.html` to the root of our project.

Add the following to `blog.html`:

{% raw %}
```html
---
layout: default
title: Blog
permalink: /blog/
---
<ul class="posts">
	{% for post in site.posts %}
		<li>
			<span>{{ post.date | date_to_string }}</span> » <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
		</li>
	{% endfor %}
</ul>
```
{% endraw %}


#### Step 14. Create a layout for posts.
As we created a `default.html` layout for our webpages, we want to create a `post.html` that will dictate how a post will be displayed.
Create a file named `post.html` in your `_layouts` folder.
Add to the file:

{% raw %}
```html
---
layout: default
---

<p class="meta">{{ page.date | date_to_string }}</p>

<div class="post">
  {{ content }}
</div>
```
{% endraw %}

#### Step 15. Create your first post
Let's test our **Blog** page!

Create a folder called `_posts` on the root of your project.
The name of the folder is mandatory so Jekyll knows where to find your posts.

Inside `_posts` create your first post with Jekyll's strict convention of `YYYY-MM-DD-title-of-your-post.md`.
For instance `2021-07-06-my-first-post.md`.

Add to your post:
```markdown
---
layout: post
title: "Today I launch my GitHub page"
date: 2021-07-06
---

# First post ever
Finally got around to getting this page up!

Using **Jekyll** I can use Markdown to author my posts.

A quick Markdown cheatsheet:
# This is the first heading
## This is the second heading.
### This is the third, and so on..

*This will appear in italic* and **this will be bold**.

We can also use bullet points:
* To enumerate things.
	* And some other things.
```

*Note that here we are using the `post` layout.*

Finally, after committing the new post navigate to your **Blog** page and click on your post.

## Wrapping up
#### Step 16. Add your own content
Right know we have three pages, **Home**, **Blog** and **Work**.

You can personalize those pages by editing `index.md`, `blog.html` and `work.md` files, respectively.
Check out how to use [Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) to create your content.
You can also create new pages.

Finally, you can create as many posts you want in your `_posts` directory as long as you use Jekyll's convention for the names of the files. As we did in [Step 15.](#step-15-create-your-first-post)


## Next Steps
This guide got me started with my own personal website and hopefully has given you the confidence to do the same.
At this point you can go in two different directions:

1. If you would prefer not to worry about the frontend aspect of your website, use a theme!
	There are several [themes](https://jekyllthemes.io/) (paid or free) available.
	Checkout Jekyll [BeautifulJekyll](https://github.com/jekyll/beautifuljekyll) free theme.
	This is the one I am using in this site.
	What you've done during this guide should be enough to understand how to navigate and add your content using a theme.

2. You would like to learn more about frontend and fully customize your website.
* Start by installing Jekyll and an IDE on your computer. I use Visual Studio Code that has great plugins for Jekyll, namely `Jekyll Run`.
This way you can change and try out things locally before committing them to your website.
* Create `_includes` to organize your markup snippets. Then you can reuse them by injecting them into your `_layouts` pages.
* Add Google Analytics to your website so you can see stats on the visitors to your website. Here's an [example](https://desiredpersona.com/google-analytics-jekyll/).
* Create a [custom 404 page](https://jekyllrb.com/tutorials/custom-404-page/).

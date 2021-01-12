# Building an Avatar Creator With Ruby On Rails and JavaScript

Almost as a guilty pleasure I’ve always loved apps that have the feature to create a custom avatar. Probably because I loved it when I unlocked a new skateboard or t-shirt on Tony Hawk’s Pro Skater and added it to my character. Upon doing research to make an avatar creator for my own rails app I found very little resources and so when I finished I decided to write a simple how to for anyone else interested. This is meant to be a guide that can be easily modified or improved upon to fit your own project. If you want to implement this in your own rails app, here are three things to consider:

-**Handling images**
-**Creating the Avatar Image form**
-**Creating and submitting base64 encoded images through the form**

##**Handling images**
Your AvatarImage is going to have an image attached that is created from the summation of all it’s smaller parts. Each of those items (head, hands, shirt, pants, feet etc…) will also have its own class and an image attached. Managing all these items means we’ll be storing, uploading and displaying lots of image files. Lucky for us, included in rails 5.2 is the Active Storage gem. 

###**Active Storage**

Active storage allows us to attach image files as well as video and audio files to a model. It’s really cool and super easy to set up. To add this to your app simply run the command:


```$ rails active_storage:install
```$ rails db:migrate





## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/AustinRhoads/building_an_avatar_creator/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/AustinRhoads/building_an_avatar_creator/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

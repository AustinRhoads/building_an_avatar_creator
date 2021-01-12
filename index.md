# Building an Avatar Creator With Ruby On Rails and JavaScript

Almost as a guilty pleasure I’ve always loved apps that have the feature to create a custom avatar. Probably because I loved it when I unlocked a new skateboard or t-shirt on Tony Hawk’s Pro Skater and added it to my character. Upon doing research to make an avatar creator for my own rails app I found very little resources and so when I finished I decided to write a simple how to for anyone else interested. This is meant to be a guide that can be easily modified or improved upon to fit your own project. If you want to implement this in your own rails app, here are three things to consider:

-**Handling images**
-**Creating the Avatar Image form**
-**Creating and submitting base64 encoded images through the form**

##**Handling images**
Your AvatarImage is going to have an image attached that is created from the summation of all it’s smaller parts. Each of those items (head, hands, shirt, pants, feet etc…) will also have its own class and an image attached. Managing all these items means we’ll be storing, uploading and displaying lots of image files. Lucky for us, included in rails 5.2 is the Active Storage gem. 

###**Active Storage**

Active storage allows us to attach image files as well as video and audio files to a model. It’s really cool and super easy to set up. To add this to your app simply run the command:


```
$ rails active_storage:install
$ rails db:migrate
```
 Once migrated, this creates the two tables active_storage_blobs and active_storage_attachments. That’s enough to get you up and running but If you want to learn more about Active Storage check out the [docs here](https://edgeguides.rubyonrails.org/active_storage_overview.html#setup).

Viola! You are now able to declare that your model has an image attached! Simply add to your models the macro has_one_attached and the attachment’s name. You can call the attachment whatever you want. Here I’ve simply called it ‘image’.

```

class Head < ApplicationRecord
	has_one_attached :image
end

class Shirt < ApplicationRecord
	has_one_attached :image
end

class Hand < ApplicationRecord
	has_one_attached :image
end

class Pant < ApplicationRecord
	has_one_attached :image
end

class Feet < ApplicationRecord
	has_one_attached :image
end
```
Make sure to add the image key to your params. To upload an image in your new and edit forms you can add a file_field to your form. 

Which looks like this in the browser:
<put image here>
  
  If your users simply want to upload a personal image as an avatar this is a great way to do that. However, we want the user to create the avatar image themselves from our own stylized characters. Now that you can upload image files for all the avatar parts, create the images as layers with a transparent background (PNG format supports transparent backgrounds). Play around with how they line up and the order of the layers. The order is important for your z-index later on. For instance you would want your pants image to be layered on top of your legs image and etc.

<put screenshot here >

To make things simple, I just stored all the item’s images in the ‘app/assets/images’ folder. Then I seeded the database with all my avatar items and attached their images like so.

```
alien_head = Head.new
alien_head.name = "Alien Head"
alien_head.image.attach(
    io: File.open("app/assets/images/avatar_demo_head_1.png"),
    filename: "nw.png"
)
alien_head.save

alien_hand = Hand.new
alien_hand.name = "Alien Hands"
alien_hand.image.attach(
    io: File.open("app/assets/images/avatar_demo_arms_1.png"),
    filename: "nw.png"
)
alien_hand.save
```
Don’t forget to run ```$ rails db:seed``` in your terminal before continuing.










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

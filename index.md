# Building an Avatar Creator With Ruby On Rails and JavaScript

Almost as a guilty pleasure I’ve always loved apps that have the feature to create a custom avatar. Probably because I loved it when I unlocked a new skateboard or t-shirt on Tony Hawk’s Pro Skater and added it to my character. Upon doing research to make an avatar creator for my own rails app I found very little resources and so when I finished I decided to write a simple how to for anyone else interested. This is meant to be a guide that can be easily modified or improved upon to fit your own project. If you want to implement this in your own rails app, here are three things to consider:

- **Handling images**
- **Creating the Avatar Image form**
- **Creating and submitting base64 encoded images through the form**

## **Handling images**
Your AvatarImage is going to have an image attached that is created from the summation of all it’s smaller parts. Each of those items (head, hands, shirt, pants, feet etc…) will also have its own class and an image attached. Managing all these items means we’ll be storing, uploading and displaying lots of image files. Lucky for us, included in rails 5.2 is the Active Storage gem. 

### **Active Storage**

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
![Image](./choose_file.png)
  
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

## **Creating the AvatarImage form**

Next we create a form to build the AvatarImage. We’ll use JavaScript and CSS to hide and display the images in real time as selected in the form.

The AvatarImage isn’t just an image but also an association of parts. We want to have the ability to call on the associated avatar items for editing our AvatarImage later on. For this the best association for the AvatarImage is that it belongs_to each avatar item.

```
class AvatarImage < ApplicationRecord
	belongs_to :user
	belongs_to :head
	belongs_to :hand
	belongs_to :shirt
	belongs_to :pant
	belongs_to :feet
	has_one_attached :image
end
```

Make collections for each item in the AvatarImage controller’s new action.

```
  def new
@avatar_image = AvatarImage.new
@heads = Head.all
@shirts = Shirt.all
@hands = Hand.all
@pants = Pant.all
@feets = Feet.all
end
```
Now we can select from each possible item in the form.

```
<%= form_for(@avatar_image, local: true, html: {multipart: true}) do |f| %>
<div class="field">
<%= f.label :head_id %>
<%= f.collection_select :head_id, @heads, :id, :name,{include_blank: 'pick your head!'}%>
</div>
<div class="field">
<%= f.label :shirt_id %>
<%= f.collection_select :shirt_id, @shirts, :id, :name,{include_blank: 'pick your shirt!'}%>
</div>
<div class="field">
<label for = "avatar_image_hand_id">Hands</label>
<%= f.collection_select :hand_id, @hands, :id, :name,{include_blank: 'pick your hands!'}%>
</div>
<div class="field">
<label for = "avatar_image_pant_id">Pants</label>
<%= f.collection_select :pant_id, @pants, :id, :name,{include_blank: 'pick your pants!'}%>
</div>
<div class="field">
<label for = "avatar_image_pant_id">Feet</label>
<%= f.collection_select :feet_id, @feets, :id, :name,{include_blank: 'pick your feet!'}%>
</div>
<%= f.hidden_field :image %>
<div class="actions">
<%= f.submit "Save avatar", :id => 'create_avatar_image' %>
</div>
<% end %>
```

This is where the CSS and JavaScript comes in. To do the real time manipulating we’ll place all of the images in the ‘views/avatar_images/_form’ file and hide them. We then add an event listener that selects the item and deselects others of it’s type. Then we add another event listener that displays that selected item’s image on a hidden canvas. Then we scale that image to avoid distortions (more on that later). Phew! Okay lets dive into it. 


In ‘views/avatar_images/_form’:

```
<% head_1 = Head.find(1) %>   
<% head_2 = Head.find(2) %>
<% head_3 = Head.find(3) %>
<%= image_tag(head_1.image, :class => "avatar_image_head_1 head hide_img") %>
<%= image_tag(head_2.image, :class => "avatar_image_head_2 head hide_img") %>
<%= image_tag(head_3.image, :class => "avatar_image_head_3 head hide_img") %>

<% hand_1 = Hand.find(1) %>
<% hand_2 = Hand.find(2) %>
<% hand_3 = Hand.find(3) %>
<%= image_tag(hand_1.image, :class => "avatar_image_hand_1 hand hide_img") %>
<%= image_tag(hand_2.image, :class => "avatar_image_hand_2 hand hide_img") %>
<%= image_tag(hand_3.image, :class => "avatar_image_hand_3 hand hide_img") %>

<% shirt_1 = Shirt.find(1) %>
<% shirt_2 = Shirt.find(2) %>
<% shirt_3 = Shirt.find(3) %>
<%= image_tag(shirt_1.image, :class => "avatar_image_shirt_1 shirt hide_img") %>
<%= image_tag(shirt_2.image, :class => "avatar_image_shirt_2 shirt hide_img") %>
<%= image_tag(shirt_3.image, :class => "avatar_image_shirt_3 shirt hide_img") %>

<% pant_1 = Pant.find(1) %>
<% pant_2 = Pant.find(2) %>
<% pant_3 = Pant.find(3) %>
<%= image_tag(pant_1.image, :class => "avatar_image_pant_1 pant hide_img") %>
<%= image_tag(pant_2.image, :class => "avatar_image_pant_2 pant hide_img") %>
<%= image_tag(pant_3.image, :class => "avatar_image_pant_3 pant hide_img") %>

<% feet_1 = Feet.find(1) %>
<% feet_2 = Feet.find(2) %>
<% feet_3 = Feet.find(3) %>
<%= image_tag(feet_1.image, :class => "avatar_image_feet_1 feet hide_img") %>
<%= image_tag(feet_2.image, :class => "avatar_image_feet_2 feet hide_img") %>
<%= image_tag(feet_3.image, :class => "avatar_image_feet_3 feet hide_img") %>
```
In your CSS file:

```
.hide_img{
    display: none;
}
```
In you’re JS file:

```

show_head();
show_feet();
show_hand();
show_shirt();
show_pants();
//grab the select element
var head_select = document.querySelector("select#avatar_image_head_id");
//add the event listeners
head_select.addEventListener("click", show_head);
head_select.addEventListener("click", construct_image);
//adds the selected class to the head image. removes the selected class from all other heads
function show_head(){
var head_select = document.getElementById("avatar_image_head_id");
if(head_select.value){
//find the image by it's id, which is the value of the select element
var head_image = document.querySelector("img.avatar_image_head_" + head_select.value);
var all_head_images = document.querySelectorAll("img.head");
//removes the class selected from all heads
all_head_images.forEach(el => el.classList.remove("selected")); 
//adds selected to the correct head
head_image.classList.add("selected");
};
};

// and now for all the rest
var hand_select = document.querySelector("select#avatar_image_hand_id");
hand_select.addEventListener("click", show_hand);
hand_select.addEventListener("click", construct_image);
function show_hand(){
var hand_select = document.getElementById("avatar_image_hand_id");
if(hand_select.value){
var hand_image = document.querySelector("img.avatar_image_hand_" + hand_select.value);
var all_hand_images = document.querySelectorAll("img.hand");
all_hand_images.forEach(el => el.classList.remove("selected"));
hand_image.classList.add("selected"); 
};
};
var shirt_select = document.querySelector("select#avatar_image_shirt_id");
shirt_select.addEventListener("click", show_shirt);
shirt_select.addEventListener("click", construct_image);
function show_shirt(){  
var shirt_select = document.getElementById("avatar_image_shirt_id");
if( shirt_select.value){
var shirt_image = document.querySelector("img.avatar_image_shirt_" + shirt_select.value);
var all_shirt_images = document.querySelectorAll("img.shirt");
all_shirt_images.forEach(el => el.classList.remove("selected"));
shirt_image.classList.add("selected");
};
};
var feet_select = document.querySelector("select#avatar_image_feet_id");
feet_select.addEventListener("click", show_feet);
feet_select.addEventListener("click", construct_image);
function show_feet(){
var feet_select = document.getElementById("avatar_image_feet_id");
if(feet_select.value){
var feet_image = document.querySelector("img.avatar_image_feet_" + feet_select.value);
var all_feet_images = document.querySelectorAll("img.feet");
all_feet_images.forEach(el => el.classList.remove("selected"));
feet_image.classList.add("selected");
};
};
var pants_select = document.querySelector("select#avatar_image_pant_id");
pants_select.addEventListener("click", show_pants);
pants_select.addEventListener("click", construct_image);
function show_pants(){
var pants_select = document.getElementById("avatar_image_pant_id");
if( pants_select.value){

var pants_image = document.querySelector("img.avatar_image_pant_" + pants_select.value);
var all_pants_images = document.querySelectorAll("img.pant");
all_pants_images.forEach(el => el.classList.remove("selected"));
pants_image.classList.add("selected");
};
};
//grab all your elements
var visible_canvas = document.querySelector("#visible_canvas");
var ctx = visible_canvas.getContext("2d");
const grid = document.querySelector(".avatar_grid");
var hidden_canvas = document.querySelector("#hidden_canvas");
var ctx_2 = hidden_canvas.getContext("2d");
// these next few functions put all the items images together as one image on the hidden canvas
//then scales that image in a way that preserves thier quality 
//to your visible canvas set to whatever dimensions you want.
function construct_image(){
ctx.clearRect(0, 0, visible_canvas.width, visible_canvas.height);
ctx_2.clearRect(0, 0, hidden_canvas.width, hidden_canvas.height);
var images_head = document.querySelector("img.head.selected");
var images_feet = document.querySelector("img.feet.selected");
var images_hand = document.querySelector("img.hand.selected");
var images_shirt = document.querySelector("img.shirt.selected");
var images_pants = document.querySelector("img.pant.selected");
var images_array = [images_shirt, images_hand, images_feet, images_head, images_pants ];
var filtered = images_array.filter(function (el) {
return el != null;
});
filtered.forEach(img => ctx_2.drawImage(img, 0,0));
hi_res_draw(hidden_canvas);
};
function hi_res_draw(img){
for(let x = 0; x < 33; x++){
var c1 = scaleIt(img,0.99999999999999999);
};
visible_canvas.width = c1.width/2;
visible_canvas.height = c1.height/2;
ctx.drawImage(c1, 0,0, c1.width, c1.height, 0, 0,  visible_canvas.width, visible_canvas.height);
};
function scaleIt(source,scaleFactor){
var c = document.createElement('canvas');
var ctx = c.getContext('2d');
var w = source.width*scaleFactor;
var h = source.height*scaleFactor;
c.width = w;
c.height = h;
ctx.drawImage(source,0,0,w,h);
return(c);
}
```

Without going into too much detail I just want to point out here that when working with canvases, if you want control over the size of the AvatarImage being displayed the canvas tends to distort when you change its size. This is why I draw the image on a hidden canvas that is the same size as the image source, resize it to scale (the scaleIt and hi_res_draw functions) and draw it to the visible canvas.

<put image of form here >

## **Creating and submitting base64 encoded images through the form**
### **Base64 encoded images**

For the Avatar’s image we are going to do something a bit different than Active Storage. As each characteristic is selected they appear on a canvas HTML element. We are going to create an image from what is visible on the canvas using JS. To accomplish this we are going to convert the visible image into an encoded Base64 image. A base64 encoded image is actually string data that represents binary data in three 8-bit bytes sequences. This encoded string can then be uploaded and converted back to an image using CarrierWave and Rmagic.

In your gemfile:

```
gem 'carrierwave-base64'
gem "rmagick"
gem "carrierwave"
```
In your terminal:

```
$ bundle update
$ rails g uploader image
$ rails g migration add_image_to_avatar_image image:string
$ rails db:migrate
```
Update your AvatarImage class by adding the following:

```
attr_accessor :remote_image_url
mount_base64_uploader :image, ImageUploader
```
Update the uploader file you created:

```
class ImageUploader < CarrierWave::Uploader::Base
 include CarrierWave::RMagick
storage :file
def store_dir
"uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
end
version :thumb do
process :resize_to_limit => [100, 100]
end
version :medium do
process :resize_to_limit => [300, 500]
end
version :large do
process :resize_to_limit => [800, 1200]
end
end
```
Now we can submit the base64 string and the uploader will convert it to an image and persist it to the database. In the ‘app/uploaders/image_uploader’ file I’ve defined three versions that will be created when an image is uploaded as well as their width and height but you can create as many versions with any name and any size you wish!

Next we can actually create the image and submit it. Here we add an event listener function to the submit button. The function ‘image_printer’ converts the visible image into dataURL (base64), creates a new PNG image with the dataURL as it’s source then submits that image in the AvatarImage form.

In your JS:
```
//grabbing the submit button and adding the function that creates the dataURL and submit it
document.getElementById('create_avatar_image').addEventListener('click', image_printer);
function image_printer(){
//converts visible to dataURL
const dataURL = hidden_canvas.toDataURL("image/png");
//creates image element and defines its source as the dataurl
var dataImg = document.createElement('img');
dataImg.src = dataURL;
var drawingField = document.createElement('div');
//this is a bit tricky!!!
drawingField.innerHTML = "<input type='hidden' name='avatar_image[image]' id='avatar_image_image' value='" + dataImg.src + "'>" ;
document.getElementById('avatar_image_image').value = dataURL;
};
```
To display it in your show page, reference the object’s image_url and specify which version you wish to use.

```
<%  if @avatar_image.image? %>
<%= image_tag @avatar_image.image_url(:medium), :class => "my_avatar_image" %>
<% end %>
```
<put image here>

And we’re done, yay! I hope this helps any interested people create thier own avatar creator. Have fun making all kinds of test avatars, and happy coding! 











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

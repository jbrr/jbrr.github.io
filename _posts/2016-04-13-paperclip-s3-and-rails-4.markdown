---
layout: post
title: "Paperclip, S3, and Rails 4"
date: 2016-04-13 12:00:00
categories: tutorials
---
Configuring Paperclip and S3 was more of a hassle a couple months ago, but things have gotten easier since the latest versions of the Paperclip gem are now compatible with the `aws-sdk` gem above versions 2.0. Nevertheless, I'll go through the steps of getting it all set up on Rails 4. The project I was working through while putting this tutorial together is [here](https://github.com/jbrr/photo-portfolio), to see all this in action.

First, we're going to need a couple of gems.

* `figaro` - We realllly don't want our AWS access keys uploaded to GitHub. Amazon is pretty good about alerting you if they do get on there somehow, but it's not worth the risk of somebody finding them and racking up huge bills for you. This is certainly possible without using Figaro, but let's just keep things simple.
* `paperclip` - As of the time of this writing, it may be useful to use the master branch as opposed to the rubygems.org version. So, we'll go ahead and use:
{% highlight ruby %}
gem "paperclip", git: "git://github.com/thoughtbot/paperclip.git"
{% endhighlight %}
* `aws-sdk` - There are certain releases that are incompatible with Paperclip, so we just want to make sure that we're using above version 2.0.34. So:
{% highlight ruby %}
gem "aws-sdk", ">= 2.0.34"
{% endhighlight %}

Second, if you don't already have it, be sure to install [ImageMagick](http://www.imagemagick.org/script/index.php). If you're not sure if you already have it installed, go ahead and run `which convert` from the command line, you should get a path back if it's installed. If you're on a Mac, that path should look something like `usr/local/bin/convert`. Take that path, without `convert`, and inside of `config/development.rb`, add `Paperclip.options[:command_path] = "/usr/local/bin/"`, or whatever your path happens to be.

Next step, is we'll go ahead and sign up for an [Amazon AWS account](https://aws.amazon.com), if you haven't already. For hobbiest projects, you shouldn't be charged. The free limit is 5 GB per month, and 20,000 GET requests. Sign up for an account, then, from your dashboard, click on your user name in the upper right corner, and click on `Security Credentials`. From there, select `Access Keys`, and click on `Create New Access Key`. Keep these keys safe.

Then, head back to your dashboard, and select `S3`. From here, we can make a new bucket. My advice to to use `US Standard`, because it allows for nested folders. Keep in mind that when it comes time to set your region in your Rails app, use `us-east-1` for `US Standard`. You can see the full list of region codes [here](http://docs.aws.amazon.com/general/latest/gr/rande.html).

Ok, now that we have our keys, we can get back to our project. My project is kind of a photograph portfolio site, so that's what this tutorial will be geared towards. If we've bundled, we can run `bundle exec figaro install`. This will create `application.yml` and append that to `.gitignore`. Inside of `application.yml`, you can go ahead and put in your access keys.

{% highlight yaml %}
production:
  AWSAccessKeyId: YOURACCESSKEY
  AWSSecretKey: YOURSECRETKEY
  S3Bucket: YOUR-BUCKET
  AWS_REGION: YOUR-REGION
development:
  AWSAccessKeyId: YOURACCESSKEY
  AWSSecretKey: YOURSECRETKEY
  S3Bucket: YOUR-BUCKET
  AWS_REGION: YOUR-REGION
test:
  AWSAccessKeyId: TEST
  AWSSecretKey: TESTSECRET
  S3Bucket: test-bucket
  AWS_REGION: YOUR-REGION
{% endhighlight %}

If we're using Heroku, production keys don't really need to be set, as those will be set inside of Heroku (since `application.yml` is in `.gitignore`, when you push to Heroku, this file won't make it up anyway). As far as test keys go, we can put in whatever we want, we'll be stubbing those responses anyway.

Ok, next step, we'll want some models, migrations, views, controllers, all that good stuff. This comes directly from the [Paperclip tutorial](https://github.com/thoughtbot/paperclip#quick-start). To keep things simple, my `Photograph` model will just have an image attachment and a title. Descriptions, user_id, EXIF data, and whatever else you might want would be trivial to add later. So:

{% highlight ruby %}
rails g model Photograph title
{% endhighlight %}

Great. Now we have a photograph model. Next, we need another migration to add the image attachment.

{% highlight ruby %}
rails g migration AddAttachmentImageToPhotographs
{% endhighlight %}

In our migration, we'll want to add an attachment:

{% highlight ruby %}
class AddAttachmentImageToPhotographs < ActiveRecord::Migration
  def self.up
    change_table :photographs do |t|
      t.attachment :image
    end
  end

  def self.down
    remove_attachment :photographs, :image
  end
end
{% endhighlight %}

We'll run the migration, and you should see a couple of new fields in your schema, such as `image_file_name`. Ok, looking good, next we'll head over to our photograph model.

{% highlight ruby %}
class Photograph < ActiveRecord::Base
  has_attached_file :image,
                    styles: { large: "800x800",
                              medium: "300x300>",
                              thumb: "100x100>" }
  validates_attachment_content_type :image, content_type: /\Aimage\/.*\Z/
end

{% endhighlight %}

Ok, next, let's take a look at our form for uploading, in `app/views/photographs/new.html.erb`:

{% highlight ruby %}
<div class="container">
  <%= render partial: "layouts/flash" %>
  <%= form_for @photograph, url: upload_path, html: { multipart: true } do |f| %>
    <%= f.label :title %>
    <%= f.text_field :title %>
    <%= f.file_field :image %>
    <%= f.submit "Submit" %>
  <% end %>
</div>
{% endhighlight %}

Ok, great, let's wire up our `photographs_controller.rb`:

{% highlight ruby %}
class PhotographsController < ApplicationController
  def new
    @photograph = Photograph.new
  end

  def create
    photograph = Photograph.create(photograph_params)
  end

  private

  def photograph_params
    params.require(:photograph).permit(:image, :title)
  end
end
{% endhighlight %}

Then, lastly, in `routes.rb`:

{% highlight ruby %}
Rails.application.routes.draw do
  resources :photographs, only: [:new, :create]
end
{% endhighlight %}

Simple enough. When it comes time to implement our show view, we'll do that with an `image_tag`:

{% highlight ruby %}
  <%= image_tag @photograph.image.url %>
{% endhighlight %}

At this point, we should be able to upload files locally. So, next step is to wire up S3. So, we need to tell the photograph model to use S3 for storage instead of saving locally. So, we'll go ahead and make some changes to `photograph.rb`:

{% highlight ruby %}
class Photograph < ActiveRecord::Base
  has_attached_file :image,
                    storage: :s3,
                    s3_region: ENV["AWS_REGION"],
                    s3_credentials: Proc.new{ |a| a.instance.s3_credentials},
                    styles: { large: "800x800",
                              medium: "300x300>",
                              thumb: "100x100>" }
  validates_attachment_content_type :image, content_type: /\Aimage\/.*\Z/

  def s3_credentials
    {bucket: ENV["S3Bucket"],
     access_key_id: ENV["AWSAccessKeyId"],
     secret_access_key: ENV["AWSSecretKey"]}
   end
end
{% endhighlight %}

One thing to keep in mind here is that your AWS region is a different argument than the credentials. Ok, at this point, we should be uploading to S3! You can go check your AWS dashboard, and look in your bucket to see if it's working.

What about testing? You've been testing, right? I've been using Minitest, and I won't walk through all of it. The model tests should be pretty straight forward. There are a couple tips for the controller tests, though.

One thing that proved to be helpful for me is `fixture_file_upload`. So, naturally, my test image is a photo of Guy Fieri looking forlorn:

![Old Guy Fieri]({{ site.url }}/assets/fieri.jpg)

So, I put that image in `test/fixtures`, and then in `photograph_controller_test.rb`:

{% highlight ruby %}
test "should upload photograph" do
    photo = fixture_file_upload("fieri.jpg", "image/jpeg")

    assert_difference("Photograph.count") do
      post(:create, {photograph: {image: photo,
                                  title: "Flavortown"}})
    end
  end
{% endhighlight %}

This is great, because now you're not uploading a file over and over every time you run a test. You're just making sure your create action is accepting the params that you want it to accept. You can do something similar to test the sad path. For this test, I have some redirect logic in my controller that I haven't explicitly walked through, and I created `test.txt` in my fixtures folder:

{% highlight ruby %}
test "should redirect to upload path if file is not image" do
    file = fixture_file_upload("test.txt", "text/plain")

    assert_no_difference("Photograph.count") do
      post(:create, {photograph: {image: file,
                                  title: "Sad Path"}})
    end

    assert_redirected_to upload_path
  end
{% endhighlight %}

Great. But, AWS isn't going to be happy with this, right? In `application.yml`, we put in some dummy access keys for the test environment, so it's not going to accept those credentials. To get around this, we can stub out the AWS response. For this, we'll go into `test_helper.rb`, and make some changes to `ActionController::TestCase`:

{% highlight ruby %}
class ActionController::TestCase
  Aws.config.update({stub_responses: true})
end
{% endhighlight %}

That should do it. We're now using Paperclip to upload files to S3, and we've got some of the tests figured out. If I missed anything, or if anyone has any suggestions, please let me know, thanks for reading!

---
layout: post
title:  "Testing view decorators using Rspec and Capybara"
date:   2014-09-06 10:15:30
categories: rspec tdd draper decorator capybara ruby on rails rspec-matchers
comments: true
---

In any web application that follows the MVC pattern, we ideally we want to keep the view templates as simple and as logic-free as possible.
The business logic should be handled between Models and Controllers and the view should just be responsible to present it.

Here is a very basic example of a view template doing more than just showing the data.

```
# => app/views/user/index.html.slim
- if @user.subscribed?
  = image_tag src="thank_you.png"
- else
  = submit_tag value="Subscribe"
```

Here we show the "thank you" image if the user is subscribed and showing something else if the user is not. All good! But how do we test this? We can't unit test this logic in the template and we don't want to write two cucumber scenarios to cover the cases where user is subscribed and not.

My answer is using a view decorator, so instead of putting the `if/else` in the template we can use <a href="https://github.com/drapergem/draper" target='_blank' > Draper </a> to create a user decorator and let this class deal with it.


```
# => app/decorators/user_decorator.rb
class UserDecorator < Draper::Decorator
  delegate_all

  def form_field_for_subscribed
    subscribed? ? thank_you_image : submit_tag
  end

  private

  def thank_you_image
    h.image_tag src="thank_you.png"
  end

  def submit_tag
    h.submit_tag value="Subscribe"
  end
end
```
Using the `user_decorator` instance instead of the `user`, now we can make the template logic-free!

```
# => app/views/user/index.html.slim
= @user.form_fields_for_subscribed

```
Now we have a class that we can test in isolation!

Great! But here is the next question: How to unit test a method that returns an HTML fragment?
I am sure there are other ways to solve this problem but this is what I did:

I wrote an rspec matcher to use Capybara and check if the fragment has the HTML nodes I expect.

```
# => spec/support/have_element_validator.rb
require 'rspec/expectations'
RSpec::Matchers.define :have_element do |type, attributes|
  match do |actual|
    result = Capybara.string(actual)
    attr_selector = attributes.inject("") do |mem, (key, value)|
      "%s[%s='%s']" % [mem, key, value]
    end
    selector = "%s%s" % [type.to_s, attr_selector]
    result.should have_selector selector
  end
end
```
So I can use this matcher to check if the decorator returned HTML fragments with the expected elements

```
# => app/spec/decorators/user_decorator_spec.rb
require 'spec_helper'

describe UserDecorator do
  describe "form_field_for_subscribed" do
    it "is the thank you image if the user is subscribed" do
      subscribed_user = FactoryGirl.create(:user, subscribed: true).decorate
      output = subscribed_user.form_field_for_subscribed
      output.should have_element :img, src: "/images/thank_you.png"
    end

    it "is the submit tag if the user is not subscribed" do
      unsubscribed_user = FactoryGirl.create(:user, subscribed: false).decorate
      output = unsubscribed_user.form_field_for_subscribed
      output.should have_element :input, type: "submit", value: "Subscribe"
    end
  end
end
```


---
layout: post
title:  "Testing Draper Decorators using Rspec and Capybara"
date:   2014-09-06 10:15:30
categories: rspec tdd draper decorator capybara ruby on rails rspec-matchers
comments: true
---

Ideally we want to keep the view templates as simple and as logic-free as possible.
Because the view layer is should only be responsible to show the data.
here is a basic example of a view template doing more than just showing the data.

```
# => app/views/user/index.html.slim
- if @user.subscribed?
  = image_tag src="thank_you.png"
- else
  = submit_tag value="Subscribe"
```


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

```
# => app/views/user/index.html.slim
= @user.form_fields_for_subscribed
```

```
# => app/spec/decorators/user_decorator_spec.rb
require 'spec_helper'

describe UserDecorator do
  describe "form_field_for_subscribed" do
    it "is the thank you image if the user is subscribed" do
      subscribed_user =  FactoryGirl.create(:user, subscribed: true).decorate
      output = subscribed_user.form_field_for_subscribed
      output.should have_element :img, src: "/images/thank_you.png"
    end

    it "is the submit tag if the user is not subscribed" do
      unsubscribed_user =  FactoryGirl.create(:user, subscribed: false).decorate
      output = unsubscribed_user.form_field_for_subscribed
      output.should have_element :input, type: "submit", value: "Subscribe"
    end
  end
end
```


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
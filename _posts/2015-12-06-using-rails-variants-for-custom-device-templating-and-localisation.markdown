---
layout: post
title:  "Using Rails Variants for Custom Device Templating and Localization"
author: mehdi
date:   2015-12-06 21:21:25
categories: web
tags: localization rails responsiveness mobile
image: /assets/article_images/2015-12-06-using-rails-variants-for-custom-device-templating-and-localisation/people-train-rails-localization-variants.jpg
---
One of the coolest features introduced in Rails 4.1 is called [ActionPack Variants]. Simply put, *request variants*
allow you to serve different templates (views) depending on the device type that is making the request.

# The Problem

Imagine the following scenario: You have a price-list view that is typically consisting of a table. Now everyone knows
that responsive tables are a conceptual nightmare for front-enders and designers. To my knowledge, no one has really yet
found a standard "widgetized" way of dealing with large numbers of columns on a mobile screen. This is probably one of
the reasons why [Foundation Responsive Tables] is still an experimental playground project off the Foundation main release track.

So you would usually go for something like this:
{% highlight html %}
  <div class="hide-for-mobile-only" id="table-container">
    <table id="price-list">
        ... Your price list...
    </table>
  </div>

  ...

  <div class="show-for-desktop-only" id="magic-widget-container">
    ... Your custom fancy price list widget ...
  </div>

{% endhighlight %}

And then tweak your declarative design back and forth to match the requirement of not displaying tables to mobile
devices but rather a convenient component of your own suited for this need.

Now you might argue that good front-enders have their own good standards and habits of solving responsive templating
design issues efficiently. But Rails isn't only about good front-enders and web-devs :) The magic of Rails lies in its
laziness. And at the core of this laziness is the universal moto of **Convention over Configuration**
# Action Pack Request Variants

Starting from Rails 4.1+, using request variants is extremely simple.
### Step 1
Add a `before_action` in your controller (logically in your `application_controller.rb` since this is an app wide
action hook). Your function could be called something like:
{% highlight ruby %}
    # Sets Action Pack variant(s) depending on the browser/device
    def set_request_variant
      request.variant = :mobile if request.user_agent =~ /android|blackberry|iphone|ipod|iemobile|mobile|webos/
    end
{% endhighlight %}

What's important here, is that the `before_action` function sets your *request variant* – an addition of Rails 4.1 to
the request.env object – to a pre-defined symbol of your own, for example `:tablet`, `:mobile` or `:desktop`
(note that you can call it whatever you wish).

What `set_request_variant` actually does is scanning the user agent object from the header of the incoming request
contained within the `request.env` object.

The annoying part here are the flags that need to be managed and written properly in order to retrieve a yes or
no answer about the type of the requesting device. Why not delegating that part to a gem dedicated to parsing user
agent data?

{% highlight ruby %}
    # Sets Action Pack variant(s) depending on the browser/device
    def set_request_variant
      request.variant = :mobile if (browser.mobile? || browser.tablet?)
    end
{% endhighlight %}

Clean and simple. Using the amazing and up-to-date [browser] gem, querying the header user agent becomes pretty
straightforward. Besides, it offers other handy querying helpers beyond device type, such us browser type, name
and media stream player type.
### Step 2
Now that our variant symbol is set (for example: `:mobile` or `:tablet`), all we have to do is include the corresponding
string to the naming of the templates we want to render to each different target. Here's the power of the *Convention*:

* `price_list.html+mobile.erb` will be rendered only to mobile devices identified by `Browser`.
* `price_list.html+tablet.erb` will conversely be rendered to tablets and only tablets.
* `price_list.html.erb` to regular desktops.

Classy isn't it?

It's worth mentioning that you can also apply the same approach to manage different browser families.

Note that the variant name ought to be appended following the `+` sign **after** the `html` extension and before `erb`.


Now what's even better, `application.html+mobile.erb` literally means you can have a different *variant* of your layout
adapted to multiple devices and clients.
# Localized Variants

The concept of variants isn't really new in Rails. It is in a certain way inferred from the notion of *localized variants*
that you can read about in the documentation of Rails' [I18n API]. As a matter of fact, localized variants are actually
quite similar to what we've been discussing so far. Instead of querying the header user agent for the display type,
we might be tempted to ask it for the browser or user language too.

The problem raising this question is quite similar. After all, ActionPack device type variants where introduced to
provide a complete control over *specific template files* to serve for each request. This takes out the burden
of unhealthy CSS and polluting our declarative code with filters and masks here and there. This is especially useful
when mobile and desktop files are **very** different content wise.

Same thing applies for languages. The I18n API provides multiple mechanisms for approaching localisation. And using
YAML files (`config -> locale --> en.yml`) is probably the most common and most adopted way.

The idea and specification requirements are quite simple:

* Get the locale from the request header
* Find out whether we want to use the user's preferred language, or if not available her browser's UI language
  (for example: A French-Canadian using an English Canadian browser)
* If session locale not already specified or different from the newly scanned locale, update the session locale from
the user agent locale
* If no locale obtained, fallback to server's locale
* Finally, add a flag switcher on the app layouts so that users can change the language whenever they want
(For example, a Spanish tourist browsing from a cyber-café in Vietnam, might need to spot the spanish flag as the only
understandable sign to her)

The problem when using localization dictionaries like YAML files resides in the case of content heavy applications.
The number of entries can grow very high and YAML files aren't meany for content anyways, only UI.
One of the greatest tools in the Rails world to localize content is the [Globalize] gem. It allows for ActiveRecord
localization which is truly a huge asset for leveraging app content management.

However, we might not be tempted to invest time, development and database maintenance for large sets of static content.
And that's where *localized variants* come in play. They work exactly like request variants described earlier, to the
difference that they use locale symbols (example: `fr` or `en`) as a convention for serving variant templates.
Hence, all you have to do is setting the locale using a regular set of `before_action` functions in the controller:

{% highlight ruby %}

 before_action :set_locale

  # Sets locale by priority depending on user preferences, geo location
  # request preferred language or default locale
  def set_locale
    I18n.locale = params[:locale] || locale_from_ui_switcher || locale_from_request_header || I18n.default_locale
  end

  # Active admin default URL locale
  def default_url_options(options={})
    {:locale => I18n.locale }
  end

  private

    # Uses the HTTP_ACCEPT_LANGUAGE header to determine the user preferred
    # rendering language (not UI language: http://www.w3.org/International/questions/qa-lang-priorities)
    def locale_from_request_header
      request.env['HTTP_ACCEPT_LANGUAGE'].scan(/^[a-z]{2}/).first
    end

    # Returns the local corresponding to the selected flag in the ui
    # upon click in the layout
    def locale_from_ui_switcher
      request.params[:locale]
    end

{% endhighlight %}


Then simply go and name your files in this way:

* `home.fr.html.erb`
* `home.html.erb`
* `my_page.es.html+mobile.erb`
* ...

As you can see, we can combine both *localized variants* and *request variants*.

You might also have noted that we added in the application controller `locale_from_ui_switcher` . This function is meant
to retrieve a locale symbol that gets appended to navigation URLs within one session by a flag/language switcher link.
Here's an example:
{% highlight html %}
<ul class="f32">
    <%= link_to content_tag(:li, '', class:'flag fr selected'), {controller: controller_name, action: action_name, locale: 'fr'} %>
</ul>
{% endhighlight%}

This link is placed for example at the top right of the application layout, meaning it will be rendered for all pages.
Thus, `controller_name` and `action_name` will automatically refer to the current page controller and action.


I hope you found this article useful in your journey with multi-lingual support in Rails! :)

*Cover Photo Source: [Nachoua]*

[ActionPack Variants]: http://guides.rubyonrails.org/4_1_release_notes.html#action-pack-variants
[Foundation Responsive Tables]: http://foundation.zurb.com/responsive-tables.html
[browser]: https://github.com/fnando/browser
[I18n API]: http://edgeguides.rubyonrails.org/i18n.html
[Globalize]: https://github.com/globalize/globalize
[Nachoua]: http://www.nachoua.com/Ph2001/Phtrainbey.htm

---
layout: post
title:  "Receiving emails in ruby on rails applications"
author: mustapha
date:   2015-05-14 11:21:25
categories: web
tags: regular email inbox rails
image: /assets/article_images/2015-05-14-receive-emails-rails/desktop.jpg
---
Ruby on Rails is well documented, easy to go web framework. But, what happens when you try developing a feature "out of the convention" that requires some advanced level of understanding of the framework?

What I kept in mind when I read Action mailer documentation at [action_mailer] was the "complex endeavor...". So, I googled "receiving emails in rails". Results are the same for you, I'm not sure but...

My web searches didn't disappoint me, I guess, Griddler does the job for most of the cases.
However, our inbox was a bit off the ordinary, especially when it comes to custom configuration, alongside our client. For example, each client needs to configure their pop3/imap parameters inside the application. Hence, We rejected Griddler method since it needs a configured mx record for the email server within the domain provider. Upset? Never I say.

Along this tutorial, we will be using [Mailman], an incoming mail processing microframework written in Ruby. First, follow the instructions and install the gem in your bundle.

# Part 1: What should be done?
Generate a new mailer with 'rails generate mailer UserMailer'. This is where messages are handled depending on your needs. In our case, we simply persisted the messages.
For this purpose, `Message` is a rails model with `body:text` and `email:string`. Put your hands up and generate the model with `rails generate message body:text email:string`.
Now, in your mailer class, define your method 'receive':

{% highlight ruby %}
def receive(message)
  message_id = message.subject[/^update (\d+)$/, 1]
	if message_id.present?
		part_to_use = message.html_part || message.text_part || message
		Message.update(message_id, body: part_to_use.body.decoded)
	else
		Message.create! subject: message.subject, body: body, email: message.from
	end
end
{% endhighlight %}

This method looks first for the message if it is present and updated it. If not, it creates a new one.
To correctly parse and encode the body, we had to add the following:
{% highlight ruby %}
 ...
else
	body = ""
	message.parts.each do |part|
	  next unless part.content_type =~ /^text\/html/
	  body << part.decode_body
	end
	Message.create! subject: message.subject, body: body.force_encoding('UTF-8'), email: message.from
end
{% endhighlight %}

## Part 2: How should it be done?
Now that we know what to do with the message, let's process it. So, under the script directory, create a script called `mailman_server.rb`:

{% highlight ruby %}

#!/usr/bin/env ruby

require 'rubygems'
require 'bundler/setup'
require 'mailman'

require File.expand_path(File.dirname(__FILE__) + '/../config/environment')

#Mailman.config.logger = Logger.new("log/mailman.log")  # uncomment this if you can want to create a log file
Mailman.config.poll_interval = 5  # change this number as per your needs. Default is 60 seconds

Mailman.config.pop3 = {
  server: 'pop.gmail.com', port: 995, ssl: true,
  username:  "GMAIL_USERNAME",
  password:  "GMAIL_PASSWORD"
}

Mailman::Application.run do
  default do
    begin
    UserMailer.receive(message)
    rescue Exception => e
      Mailman.logger.error "Exception occurred while receiving message:n#{message}"
      Mailman.logger.error [e, *e.backtrace].join("n")
    end
  end
end
{% endhighlight %}

As an example, we used gmail pop3 configuration. If you decide to go with imap, don't forget to change the settings in your Gmail for redirection.
Inside the `Mailman::Application`, lies what we call `routes`.
Inside `Default` route, we have access to the `message` object which is an instance of the `Mailman::Message`. Then, We can call our behaviour from the `UserMailer` method `receive`.
It is time to test it out by running in the console `bundle exec mailman_server.rb`. You should see something like:

```
I, INFO -- : Mailman v0.7.3 started
I, INFO -- : POP3 receiver enabled (stoufa.turki@gmail.com@pop.gmail.com).
I, INFO -- : Polling enabled. Checking every 5 seconds.
I, INFO -- : Got new message from 'username@gmail.com' with subject 'Subject'.
```

## Part 3: What if this is not exactly what we want?
All what we have seen previously was blogged before and screen-casted here at [mailman-webcast].

Let's discuss what we want. We want freedom. We want every client of the application (for example a saas) be able to configure their own email settings. To do this, we created a new model `configs` where all configuration (server, username, password_encrypted...) and the corespondent user is held.
Look at the password column, how do you feel? Scared, Hah! Don't worry.
Generate the model for now and keep it going like a waterfall.

Regarding the encryption, we used symetric encryption `openssl` to encrypt/decrypt passwords. This cheatcode is a good place to understand: [openssl-cheat].
So to encrypt the password, we defined the following helper:

{% highlight ruby %}

require 'openssl'
require 'base64'

module Mailer
	module SettingsHelper

		def encypt_password(password)
			cipher = OpenSSL::Cipher::AES.new(128, :CBC)
			cipher.encrypt
			cipher.key = Rails.application.secrets.secret_ssl
			password_encryped = cipher.update(params[:password]) + cipher.final
			Base64.strict_encode64 password_encryped
		end

	end
end
{% endhighlight %}
But what is the `Base64` doing there? It encodes the binary data after encryption to return a ready to persist data format.
Regarding `Rails.application.secrets.secret_ssl`, it is the new way to store secrets in `Rails 4.2`. For more details, see [rails-upgrade]

Our helper is ready, we can use it within the controller:

{% highlight ruby %}
	config.password_encrypted = encypt_password(params[:password_encryped])
{% endhighlight %}

Now that we have a configuration, we can retrieve it in our server and set it as the config for the `Mailman::Application`.
And as we did for the encryption, we did for the decryption using the same `secret`.
{% highlight ruby %}
...
BEGIN {
  def decrypt_password(password_encrypted)
    decipher = OpenSSL::Cipher::AES.new(128, :CBC)
    decipher.decrypt
    decipher.key = Rails.application.secrets.secret_ssl
    password_encrypted = Base64.strict_decode64 password_encrypted
    decrypted_password = decipher.update(password_encrypted) + decipher.final
  end

  def inbox_config
    Config.find_by user_id: ARGV[0]
  end
}

Mailman::Application.run do
  initialize do
    Mailman.config.pop3 = {
      server: 'pop.gmail.com', port: 995, ssl: true,
      username: inbox_config.username,
      password: decrypt_password(inbox_config.password_encrypted)
    }
  end
	...
end
{% endhighlight %}

`ARGV[0]` stands for the `user_id` parameter passed when starting the server.
By the end, the complete `mailman_server.rb`:

{% highlight ruby %}

#!/usr/bin/env ruby

require 'rubygems'
require 'bundler/setup'
require 'mailman'
require 'bcrypt'
require "openssl"
require "base64"

require File.expand_path(File.dirname(__FILE__) + '/../config/environment')

#Mailman.config.logger = Logger.new("log/mailman.log")  # uncomment this if you can want to create a log file
Mailman.config.poll_interval = 5  # change this number as per your needs. Default is 60 seconds

BEGIN {
  def decrypt_password(password_encrypted)
    decipher = OpenSSL::Cipher::AES.new(128, :CBC)
    decipher.decrypt
    decipher.key = Rails.application.secrets.secret_ssl
    password_encrypted = Base64.strict_decode64 password_encrypted
    decrypted_password = decipher.update(password_encrypted) + decipher.final
  end

  def inbox_config
    Config.find_by user_id: ARGV[0]
  end
}

Mailman::Application.run do
  initialize do
    Mailman.config.pop3 = {
      server: 'pop.gmail.com', port: 995, ssl: true,
      username: inbox_config.username,
      password: decrypt_password(inbox_config.password_encrypted)
    }
  end
  default do
    begin
    UserMailer.receive(message)
    rescue Exception => e
      Mailman.logger.error "Exception occurred while receiving message:n#{message}"
      Mailman.logger.error [e, *e.backtrace].join("n")
    end
  end
end
{% endhighlight %}

Start the server using the command `bundle exec script/mailman_server.rb param1`. param> is the `user_id` for the used configuration.

You can also create a daemon for running the server in the background by creating a new file called `mailman_daemon.rb`:

{% highlight ruby %}

#!/usr/bin/env ruby
require 'daemons'

Daemons.run 'script/mailman_server.rb'
{% endhighlight %}

After installing the daemons gem, you are ready to go with `bundle exec script/mailman_daemon.rb start -- param1`. Note that the dashed separation before the params is important.

##Part 4: Cheers

[mailman]: 		 https://github.com/mailman/mailman
[action_mailer]: 		 http://guides.rubyonrails.org/action_mailer_basics.html#receiving-emails
[griddler]: https://github.com/thoughtbot/griddler
[mailman-webcast]: http://railscasts.com/episodes/313-receiving-email-with-mailman?view=comments
[openssl-cheat]: https://github.com/augustl/ruby-openssl-cheat-sheet/blob/master/encryption_symmetric.rb
[rails-upgrade]: http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html#config-secrets-yml

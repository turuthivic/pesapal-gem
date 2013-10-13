Pesapal RubyGem
===============

<a href="http://badge.fury.io/rb/pesapal"><img src="https://badge.fury.io/rb/pesapal@2x.png" alt="Gem Version" height="18"></a>

Make authenticated Pesapal API calls without the fuss! Handles all the [oAuth
stuff][1] abstracting any direct interaction with the API endpoints so that you
can focus on what matters. _Building awesome_.

This gem is work in progress. At the moment, the only functionality built-in is
posting an order i.e. fetching the URL that is required to display the post-
order iframe. Everything else should be easy to do as the groundwork has already
been laid. If you are [feeling generous and want to contribute, feel free][9].

Submit [issues and requests here][6] and [find all the releases here][12].

The gem should be [up on RubyGems.org][7] and it's [accompanying RubyDoc reference here][13].

_Ps: No 3rd party oAuth library dependencies, it handles all the oAuth flows on
it's own so your app is one dependency less._

_Ps 2: We are still at pre-release stage ... target is version 1.0.0 for a
public release (suitable for production deployment with the basic functionality
in place). As a result always check the documentation carefully on upgrades to
mitigate breaking changes._


Installation
------------

Add this line to your application's Gemfile:

    gem 'pesapal'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install pesapal

For Rails, you need to run the generator to set up some necessary stuff:

    rails generate pesapal:install


Usage
-----


### Initialization ###

There are 3 ways to initialize the Pesapal object:

1. YAML config at `/config/pesapal.yml` (default & recommended)
2. YAML config at custom location
3. Config hash

Initialize Pesapal object and choose the mode, there are two modes;
`:development` and `:production`. They determine if the code will interact
with the testing or the live Pesapal API.

```ruby
# initiate pesapal object set to development mode
pesapal = Pesapal::Merchant.new(:development)
```

####Option 1####

In the case, the configuration has already been loaded (at application start by
initializer) from a YAML file located at `"#{Rails.root}/config/pesapal.yml"` by
default. This is the recommended method.

####Option 2####

It's also possible to set the configuration details from a YAML file at the
location of your choice upon initialization as shown in the example below. This
option overrides the default YAML config (explained above) and will be loaded
everytime during initialization (which is not a good idea for production).

```ruby
# initiate pesapal object set to development mode and use the YAML file found at
# the specified location ... this overrides and loads the YAML file afresh
pesapal = Pesapal::Merchant.new(:development, "<PATH_TO_YAML_FILE>")
```

####Option 3####

If you wish not to use the YAML config method or the YAML file for some reason
does not exist, then the object is set up with some bogus credentials which
would not work anyway and therefore, the other option is that you set them
yourself. Which, you can do using a hash as shown below (please note that
Pesapal provides different keys for different modes and since this is like an
override, there's the assumption that you chose the right one).

```ruby
# set pesapal api configuration manually (override YAML & bogus credentials)
pesapal.config = {  :callback_url => 'http://0.0.0.0:3000/pesapal/callback'
                    :consumer_key => '<YOUR_CONSUMER_KEY>',
                    :consumer_secret => '<YOUR_CONSUMER_SECRET>'
                  }
```


###YAML Configuration###

The YAML file should look something like this. If you ran the generator you
should have it already in place with some default values. Feel free to change
them appropriately.

```yaml
development:
    callback_url: 'http://0.0.0.0:3000/pesapal/callback'
    consumer_key: '<YOUR_CONSUMER_KEY>'
    consumer_secret: '<YOUR_CONSUMER_SECRET>'

production:
    callback_url: 'http://1.2.3.4:3000/pesapal/callback'
    consumer_key: '<YOUR_CONSUMER_KEY>'
    consumer_secret: '<YOUR_CONSUMER_SECRET>'
```


### Posting An Order ###

Once you've finalized the configuration, set up the order details in a hash as
shown in the example below ... all keys **MUST** be present. If there's one that
you wish to ignore just leave it with a blank string but make sure it's included
e.g. the phonenumber.

```ruby
#set order details 
pesapal.order_details = { :amount => 1000,
                          :description => 'this is the transaction description',
                          :type => 'MERCHANT',
                          :reference => 808-707-606,
                          :first_name => 'Swaleh',
                          :last_name => 'Mdoe',
                          :email => 'user@example.com',
                          :phonenumber => '+254722222222'
                        }
```

Then generate the transaction url as below. In the example, the value is
assigned to the variable `order_url` which you can pass on to the templating
system of your choice to generate an iframe. Please note that this method
utilizes all that information set in the previous steps in generating the url so
it's important that it's the last step in the post order process.

```ruby
# generate transaction url
order_url = pesapal.generate_order_url

# order_url will a string with the url example;
# http://demo.pesapal.com/API/PostPesapalDirectOrderV4?oauth_callback=http%3A%2F%2F1.2.3.4%3A3000%2Fpesapal%2Fcallback&oauth_consumer_key=A9MXocJiHK1P4w0M%2F%2FYzxgIVMX557Jt4&oauth_nonce=13804335543pDXs4q3djsy&oauth_signature=BMmLR0AVInfoBI9D4C38YDA9eSM%3D&oauth_signature_method=HMAC-SHA1&oauth_timestamp=1380433554&oauth_version=1.0&pesapal_request_data=%26lt%3B%3Fxml%20version%3D%26quot%3B1.0%26quot%3B%20encoding%3D%26quot%3Butf-8%26quot%3B%3F%26gt%3B%26lt%3BPesapalDirectOrderInfo%20xmlns%3Axsi%3D%26quot%3Bhttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema-instance%26quot%3B%20xmlns%3Axsd%3D%26quot%3Bhttp%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%26quot%3B%20Amount%3D%26quot%3B1000%26quot%3B%20Description%3D%26quot%3Bthis%20is%20the%20transaction%20description%26quot%3B%20Type%3D%26quot%3BMERCHANT%26quot%3B%20Reference%3D%26quot%3B808%26quot%3B%20FirstName%3D%26quot%3BSwaleh%26quot%3B%20LastName%3D%26quot%3BMdoe%26quot%3B%20Email%3D%26quot%3Bj%40kingori.co%26quot%3B%20PhoneNumber%3D%26quot%3B%2B254722222222%26quot%3B%20xmlns%3D%26quot%3Bhttp%3A%2F%2Fwww.pesapal.com%26quot%3B%20%2F%26gt%3B
```

_Ps: Please note the `:callback_url` value in the `pesapal.config` hash ...
after the user successfully posts the order, the response will be sent to this
url. Refer to [official Pesapal Step-By-Step integration guide][18] for more
details._


### Querying Payment Status ###

Use this to query the status of the transaction. When a transaction is posted to
Pesapal, it may be in a PENDING, COMPLETED, FAILED or INVALID state. If the
transaction is PENDING, the payment may complete or fail at a later stage.

Both the unique merchant reference generated by your system (compulsory) and the
pesapal transaction tracking id (optional) are input parameters to this method
but if you don't ensure that the merchant reference is unique for each order on
your system, you may get INVALID as the response. Because of this, it is
recommended that you provide both the merchant reference and transaction
tracking id as parameters to guarantee uniqueness.

```ruby
# option 1: using merchant reference only
payment_status = pesapal.query_payment_status("<MERCHANT_REFERENCE>")

# option 2: using merchant reference and transaction id (recommended)
payment_status = pesapal.query_payment_status("<MERCHANT_REFERENCE>","<TRANSACTION_ID>")
```


Contributing
------------

1. Make sure you've read the [M.O. ★][14] ([blog article here][16])
2. Especially [the part about my conventions when writing and merging new features][15]
2. [Fork it][8]
2. Create your feature branch (`git checkout -b BRANCH_NAME`)
3. Commit your changes (`git commit -am 'AWESOME COMMIT MESSAGE'`)
4. Push to the branch (`git push origin BRANCH_NAME`)
5. Create new pull request and we can [have the conversations here][17]

_Ps: See [current contributors][19]._


References
----------

* [oAuth 1.0 Spec][1]
* [Developing a RubyGem using Bundler][2]
* [RailsGuides][20]
* [Make your own gem][3]
* [Pesapal API Reference (Official)][4]
* [Pesapal Step-By-Step Reference (Official)][18]
* [Pesapal PHP API Reference (Unofficial)][5]


License
-------

[King'ori J. Maina][10] © 2013. The [MIT License bundled therein][11] is a
permissive license that is short and to the point. It lets people do anything
they want as long as they provide attribution and waive liability.

[1]: http://oauth.net/core/1.0/
[2]: https://github.com/radar/guides/blob/master/gem-development.md
[3]: http://guides.rubygems.org/make-your-own-gem/
[4]: http://developer.pesapal.com/how-to-integrate/api-reference
[5]: https://github.com/itsmrwave/pesapal-php#pesapal-php-api-reference-unofficial
[6]: https://github.com/itsmrwave/pesapal-rubygem/issues
[7]: http://rubygems.org/gems/pesapal
[8]: https://github.com/itsmrwave/pesapal-rubygem/fork
[9]: https://github.com/itsmrwave/pesapal-rubygem#contributing
[10]: http://kingori.co/
[11]: https://github.com/itsmrwave/pesapal-rubygem/blob/master/LICENSE.txt
[12]: https://github.com/itsmrwave/pesapal-rubygem/releases/
[13]: http://rubydoc.info/gems/pesapal/
[14]: https://github.com/itsmrwave/mo
[15]: https://github.com/itsmrwave/mo/tree/master/convention#-convention
[16]: http://kingori.co/articles/2013/09/modus-operandi/
[17]: https://github.com/itsmrwave/pesapal-rubygem/pulls
[18]: http://developer.pesapal.com/how-to-integrate/step-by-step
[19]: https://github.com/itsmrwave/pesapal-rubygem/graphs/contributors
[20]: http://guides.rubyonrails.org/

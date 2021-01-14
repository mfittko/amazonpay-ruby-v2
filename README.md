# Ruby version Amazon Pay v2
The following is a Ruby version sample application of Amazon Pay v2.
http://amazonpaycheckoutintegrationguide.s3.amazonaws.com/amazon-pay-checkout/introduction.html

## Operating environment
Ruby 2.3.0 or higher
Note: If it is less than 2.5.0, you need to update the "openssl" gem according to the instructions below.
https://github.com/ruby/openssl

## Overview
This application provides a sample that executes a simple purchase flow with Amazon Pay as shown below.

<img src = "images / checkout-flow.gif" width = "350">

## Install

### Repository clone
Clone this repository.
```sh
git clone https://github.com/amazonpay-labs/amazonpay-sample-ruby-v2.git

#Move to cloned repository
cd amazonpay-sample-ruby-v2
```

### Application creation / setting in Seller Central
Execute the Ruby script with the following command.
```sh
ruby keys / init.rb
```

The following files will be generated under the keys directory.
  --keyinfo.rb
  --privateKey.pem

Prepare an application for this sample at [Seller Central] (https://sellercentral.amazon.co.jp/) and [here] (https://amazonpaycheckoutintegrationguide.s3.amazonaws.com/amazon-pay- Get Merchant ID, Public Key ID, Store ID, Private Key by referring to checkout / get-set-up-for-integration.html # 4-get-your-public-key-id) and copy them below. To do.
  * Merchant ID: keys / keyinfo.rb merchant_id
  * Public Key ID: public_key_id in keys / keyinfo.rb
  * Store ID: store_id in keys / keyinfo.rb
  * Private Key: keys / privateKey.pem

### Installation of dependent modules
In this directory, execute the following command to install the dependent modules.

#### When using Bundler
```sh
bundle install
```

#### When using RubyGems
```sh
# Note! ↓ ↓ ↓ Only if the Ruby version is less than 2.5.0 ↓ ↓ ↓
gem install openssl
# Note! ↑↑↑ Only if the Ruby version is less than 2.5.0 ↑↑↑
# Also, as described in https://github.com/ruby/openssl, in case of 2.3, you also need to execute "gem'openssl'" in the source to enable gem.
# In this application, this process is executed at the beginning of the code in libs / signature.rb.

gem install sinatra
```

## Start the server
Execute the following command in this directory.
```sh
ruby app.rb
```

Go to [http: // localhost: 4567 /] (http: // localhost: 4567 /) and check the operation.

# Configuration of this application

This application mainly consists of the following three rb files.

## app.rb
The main body of the web application. It is implemented in [Sinatra] (http://sinatrarb.com/) and has about 100 lines.

## keys / keyinfo.rb
This is a configuration file in which only various setting values ​​are defined.

## libs / signature.rb
A sample showing how to call the Amazon Pay API, with about 120 lines of code.
The usage of the sample code is shown in English at the beginning of the file, and the following is the Japanese translation.

---

First, instantiate the Amazon Pay Client as shown below. ::
```ruby
    client = AmazonPayClient.new {
        public_key_id:'XXXXXXXXXXXXXXXXXXXXXXXX', publick key ID obtained from #SellerCentral
        private_key: File.read ('./ privateKey.pem'), private key obtained from #SellerCentral
        region:'jp', #'na','eu','jp' can be specified
        sandbox: true
    }
```

### When generating a button signature
Call'generate_button_signature'with the following parameters.
 --payload: Payload to pass to the API. It can be a JSON string or a Hash instance.

See also: http://amazonpaycheckoutintegrationguide.s3.amazonaws.com/amazon-pay-checkout/add-the-amazon-pay-button.html#3-sign-the-payload

Example:
```ruby
    signature = client.generate_button_signature {
        webCheckoutDetails: {
            checkoutReviewReturnUrl:'http://example.com/review'
        },
        storeId:'amzn1.application-oa2-client.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
    }
```

### Other API calls

Call'call_api'with the following parameters specified.
 --url_fragment: The last part of the URL of the API call. Example) In the case of'https://pay-api.amazon.com/:environment/:version/checkoutSessions/', "checkoutSessions"
 --method: HTTP method of API call
 -(Optional) payload: Request payload for API calls. It can be a JSON string or a Hash instance.
 -(Optional) headers: HTTP headers for API calls. Example) {header1:'value1', header2:'value2'}
 -(Optional) query_params: query parameter for API calls. Example) {param1:'value1', param2:'value2'}
 The response of the API call is returned.

Example 1: [Create Checkout Session] (http://amazonpaycheckoutintegrationguide.s3.amazonaws.com/amazon-pay-api-v2/checkout-session.html#create-checkout-session)

```ruby
    response = client.api_call ("checkoutSessions", "POST",
        payload: {
            webCheckoutDetails: {
                checkoutReviewReturnUrl: "https://example.com/review"
            },
            storeId: "amzn1.application-oa2-client.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        },
        headers: {'x-amz-pay-idempotency-key': SecureRandom.hex (10)}
    )
```

Example 2: [Get Checkout Session] (http://amazonpaycheckoutintegrationguide.s3.amazonaws.com/amazon-pay-api-v2/checkout-session.html#get-checkout-session)

```ruby
    response = client.api_call ("checkoutSessions / # {amazon_checkout_session_id}",'GET')
```

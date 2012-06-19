# shopify_client

Simple [Shopify API](http://api.shopify.com/) client in PHP (currently only works with the new oAuth2 version)


## Requirements

* PHP 5.3 with [cURL support](http://php.net/manual/en/book.curl.php).


## Getting Started

### Download

#### Via [Composer](http://getcomposer.org/)

This is the preferred method as it will also download dependencies.

Create a `composer.json` file if you don't already have one in your projects root directory and require shopify_client:

```
{
	"require": {
		"sandeepshetty/shopify_client": "dev-master"
	}
}
```

Install Composer:
```
$ curl -s http://getcomposer.org/installer | php
```

Run the install command:
```
$ php composer.phar install
```

This will download shopify_client into the `vendor/sandeepshetty/shopify_client` directory.

To learn more about Composer visit http://getcomposer.org/


#### Via an archive

Download the [latest version of shopify_client](https://github.com/sandeepshetty/shopify_client/archives/master):

```shell
$ curl -L http://github.com/sandeepshetty/shopify_client/tarball/master | tar xvz
$ mv sandeepshetty-shopify_client-* shopify_client
```
Since shopify_client requires [wcurl](https://github.com/sandeepshetty/wcurl), you'll have to download that manually as well.


### Require

```php
<?php

	require 'vendor/sandeepshetty/shopify_client/shopify_client.php';

?>
```

### Usage

#### Generating the app's installation URL for a given store:
```php
<?php

	$install_url = install_url($shop_domain, $api_key);
?>
```

#### Verify the origin of all requests/redirects from Shopify:
```php
<?php

	if (is_valid_request($_GET, $shared_secret))
	{
		...
	}
?>
```

#### Generating the oAuth2 permission URL:
```php
<?php

	$permission_url = permission_url($_GET['shop'], $api_key, array('read_products', 'read_orders'));

?>
```

#### Get the permanent oAuth2 access token:
```php
<?php

	$access_token = oauth_access_token($_GET['shop'], $api_key, $shared_secret, $_GET['code'])

?>
```

#### Making API calls:

```php
<?php

	// For regular apps:
	$shopify = shopify_client($shops_myshopify_domain, $shops_access_token, $api_key, $shared_secret);

	// For private apps:
	// $shopify = shopify_client($shops_myshopify_domain, NULL, $api_key, $password, true);

	// If your migrating from legacy auth:
	// $password = legacy_token_to_oauth_token($shops_legacy_token, $shared_secret);
	// $shopify = shopify_client($shops_myshopify_domain, $password, $api_key, $shared_secret);

	try
	{
		// Get all products
		$products = $shopify('GET', '/admin/products.json', array('published_status'=>'published'));


		// Create a new recurring charge
		$charge = array
		(
			"recurring_application_charge"=>array
			(
				"price"=>10.0,
				"name"=>"Super Duper Plan",
				"return_url"=>"http://super-duper.shopifyapps.com",
				"test"=>true
			)
		);

		try
		{
			// All requests accept an optional fourth parameter, that is populated with the response headers.
			$recurring_application_charge = $shopify('POST', '/admin/recurring_application_charges.json', $charge, $response_headers);

			// API call limit helpers
			echo shopify_calls_made($response_headers); // 2
			echo shopify_calls_left($response_headers); // 298
			echo shopify_call_limit($response_headers); // 300

		}
		catch (ShopifyClientApiException $e)
		{
			// If you're here, either HTTP status code was >= 400 or response contained the key 'errors'
		}

	}
	catch (ShopifyClientApiException $e)
	{
		/* $e->getInfo() will return an array with keys:
			* method
			* path
			* params (third parameter passed to $shopify)
			* response_headers
			* response
			* shops_myshopify_domain
			* shops_token
		*/
	}
	catch (ShopifyClientCurlException $e)
	{
		// $e->getMessage() returns value of curl_ error() and $e->getCode() returns value of curl_errno()
	}
?>
```

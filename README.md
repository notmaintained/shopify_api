# shopify_api

Simple [Shopify API](http://api.shopify.com/) client in PHP


## Requirements

* PHP 5.3 with [cURL support](http://php.net/manual/en/book.curl.php).


## Getting Started

### Download via [Composer](http://getcomposer.org/)

Create a `composer.json` file if you don't already have one in your projects root directory and require shopify_api:

```
{
	"minimum-stability": "dev",
	"require": {
		"sandeepshetty/shopify_api": "dev-master"
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

This will download shopify_api into the `vendor/sandeepshetty/shopify_api` directory.

To learn more about Composer visit http://getcomposer.org/


### Require and use

```php
<?php

	require 'vendor/sandeepshetty/shopify_api/client.php';
	use sandeepshetty\shopify_api;

?>
```

### Usage

#### Generating the app's installation URL for a given store:
```php
<?php

	$install_url = shopify_api\install_url($shop_domain, $api_key);
?>
```

#### Verify the origin of all requests/redirects from Shopify:
```php
<?php

	if (shopify_api\is_valid_request($_GET, $shared_secret))
	{
		...
	}
?>
```

#### Generating the oAuth2 permission URL:
```php
<?php

	$permission_url = shopify_api\permission_url($_GET['shop'], $api_key, array('read_products', 'read_orders'));

?>
```

#### Get the permanent oAuth2 access token:
```php
<?php

	$access_token = shopify_api\oauth_access_token($_GET['shop'], $api_key, $shared_secret, $_GET['code'])

?>
```

#### Making API calls:

```php
<?php

	// For regular apps:
	$shopify = shopify_api\client($shops_myshopify_domain, $shops_access_token, $api_key, $shared_secret);

	// For private apps:
	// $shopify = shopify_api\client($shops_myshopify_domain, NULL, $api_key, $password, true);

	// If your migrating from legacy auth:
	// $password = shopify_api\legacy_token_to_oauth_token($shops_legacy_token, $shared_secret);
	// $shopify = shopify_api\client($shops_myshopify_domain, $password, $api_key, $shared_secret);

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
			echo shopify_api\calls_made($response_headers); // 2
			echo shopify_api\calls_left($response_headers); // 298
			echo shopify_api\call_limit($response_headers); // 300

		}
		catch (shopify_api\Exception $e)
		{
			// If you're here, either HTTP status code was >= 400 or response contained the key 'errors'
		}

	}
	catch (shopify_api\Exception $e)
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
	catch (shopify_api\CurlException $e)
	{
		// $e->getMessage() returns value of curl_error() and $e->getCode() returns value of curl_errno()
	}
?>
```

# Description

[![Latest Version on Packagist](https://img.shields.io/packagist/v/frog/php-curl-sai.svg?style=flat-square)](https://packagist.org/packages/frog/php-curl-sai)
[![Build Status](https://img.shields.io/travis/frog/php-curl-sai/master.svg?style=flat-square)](https://travis-ci.com/github/frogtechnologies/php-curl-sai)
[![Total Downloads](https://img.shields.io/packagist/dt/frog/php-curl-sai.svg?style=flat-square)](https://packagist.org/packages/frog/php-curl-sai)

This is a minimal package forked from [m-ender/php-SAI](https://github.com/m-ender/php-SAI) aimed at enabling a simplified approach to stubbing of curl requests in php projects. The main changes it underwent were:

-   Removal of unnecessary code files such as multi curl
-   Upgrading of curl options as the last update the package received was in 2012

## Installation

You can install the package via composer:

```bash
composer require frog/php-curl-sai:1.0.0
```

## Usage

In the class that's responsible for making the request:

```php

use FROG\PhpCurlSAI\SAI_Curl;
use FROG\PhpCurlSAI\SAI_CurlInterface;

 class SDKClass {
    // Define the variable to hold the cURL instance that will make a connection
    protected $cURL;

    /** Pass in the interface implemented by both the stub and actual class
     * to allow for detection of methods to be used by your IDE
    **/
    public function __construct(
        SAI_CurlInterface $cURL = null
    ) {
        // If no curl instance is provided, the one that makes the actual connection shall be used
        if ($cURL == null) $this->cURL = new SAI_Curl();
        else
            $this->cURL = $cURL;
    }

    public function get_data(
        string $access_token
    ){
        $auth_headers = [
            "Authorization: Bearer {$access_token}",
            "Content-Type: application/json",
        ];

        $options = [
            CURLOPT_URL => 'https://cool.url/data',
            CURLOPT_HTTPHEADER => $auth_headers,
            CURLOPT_SSL_VERIFYPEER  => false,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POST => true,
        ];

        // Setup a cURL handle via the interface
        $ch = $this->cURL->curl_init();

        // Add curl options via the interface
        $this->cURL->curl_setopt_array($ch, $options);
        // Execute the reqeust via the interface
        $response = $this->cURL->curl_exec($ch);

        if ($response === false) {
            // Retrieve the error via the interface
            $result = $this->cURL->curl_error($ch);
        } else {
            // Decode the result if need be
            $result = json_decode($response);
        }

        // Close the handle via the interface
        $this->cURL->curl_close($ch);

        return $result;
    }

    public function authorize(
        string $email,
        string $password,
    ){
        $request_body = [
            'email' => 'doe@mail.com',
            'password' => "12345678",
        ];

        $options = [
            CURLOPT_URL => 'https://cool.url/token',
            CURLOPT_HTTPHEADER => ["Content-Type: application/json"],
            CURLOPT_SSL_VERIFYPEER  => false,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => json_encode($request_body),
        ];

        // Setup a cURL handle via the interface
        $ch = $this->cURL->curl_init();

        // Add curl options via the interface
        $this->cURL->curl_setopt_array($ch, $options);
        // Execute the reqeust via the interface
        $response = $this->cURL->curl_exec($ch);

        if ($response === false) {
            // Retrieve the error via the interface
            $result = $this->cURL->curl_error($ch);
        } else {
            // Decode the result if need be
            $result = json_decode($response);
        }

        // Close the handle via the interface
        $this->cURL->curl_close($ch);

        return $result;
    }
 }

```

In the class/file that wants to perform a stub, e.g during tests:

```php
use FROG\PhpCurlSAI\SAI_CurlStub;

public function stub_get_data() {
    // Create an instance of the stub class which implements SAI_CurlInterface
    $cURL = new SAI_CurlStub();
    $sdk = new SDKClass($cURL);

    $request_body = [
        'email' => 'doe@mail.com',
        'password' => "12345678",
    ];

    $options = [
        CURLOPT_URL => 'https://cool.url/token',
        CURLOPT_HTTPHEADER => ["Content-Type: application/json"],
        CURLOPT_SSL_VERIFYPEER  => false,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => json_encode($request_body),
    ];

    // Set the response you want returned when the first cURL call is to be made
    $cURL->setResponse(
        '{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c","token_type":"Bearer","expires_in":3600}',
        // Pass in the cURL options (Needed to get the desired output)
        $options,
    );

    $token_result = $sdk->authorize(
      'doe@mail.com',
      "12345678"
    );

    $auth_headers = [
        "Authorization: Bearer {$token_result->access_token}",
        "Content-Type: application/json",
    ];

    $options = [
        CURLOPT_URL => 'https://cool.url/data',
        CURLOPT_HTTPHEADER => $auth_headers,
        CURLOPT_SSL_VERIFYPEER  => false,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
    ];

    // Set the response you want returned when the second cURL call is to be made
    $cURL->setResponse(
        '{"ref":"ac980b8e-afe5-49b8-b348-d0af00e2f556","date":"2021-02-21 22:20:32","code":"-11","MessageDescription":"MESSAGE REFERENCE LONGER THAN ALLOWED LENGTH"}',
        // Pass in the cURL options (Needed to get the desired output)
        $options,
    );

    $result = $sdk->get_data($token_result->access_token);

}

```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email milleradulu@gmail.com instead of using the issue tracker.

## Credits

-   [Miller Adulu](https://github.com/frog)
-   [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

## PHP Package Boilerplate

This package was generated using the [PHP Package Boilerplate](https://laravelpackageboilerplate.com).

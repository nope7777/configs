# About

![GitHub Workflow Status](https://img.shields.io/github/workflow/status/nope7777/configs/common-workflow)

Usually configuration files in PHP projects stored in PHP files as arrays or in YAML files. 
I see few cons in this approach:
- In case you need some configurations in your service, you must inject all configurations.
- Structure on those configurations not amenable to static analysis.

This small library offers a slightly different approach for organizing your configuration files.

Usual configuration file:

```php
<?php

return [
    'application' => [
        'timezone' => env('APPLICATION_TIMEZONE', 'UTC'),
        'currency' => env('APPLICATION_CURRENCY', 'USD'),
    ],
    'storage' => [
        'driver' => env('STORAGE_DRIVER', 'local'),
        'drivers' => [
            'aws' => [
                // ...
            ],
            'local' => [
                // ...
            ],
        ],
    ],
];
```

Used like this:

```php
<?php

class LocalStorageAdapter
{
    private Config $config;
    
    public function __construct(Config $config)
    {
        $this->config = $config;
    }
    
    public function build()
    {
        // IDE's autocomplete not working for this line
        $this->config->storage->drivers->local;
    }
}
```

Can be transformed in to:

```php
use N7\Configs\AbstractConfig;

class ApplicationConfiguration extends AbstractConfig
{
    private string $timezone;
    private string $currency;
    
    public function __construct()
    {
        $this->timezone = $this->env->getString('APPLICATION_TIMEZONE', 'UTC', false);
        $this->currency = $this->env->getString('APPLICATION_CURRENCY', 'USD', false);
    }
}

class StorageConfiguration extends AbstractConfig
{
    private string $driver;
    
    private array $drivers = [];
    
    // If you want you are able to inject nested configurations
    public function __construct(
        LocalStorageConfiguration $localStorageConfiguration,
        AwsStorageConfiguration $awsStorageConfiguration
    ) {
        $this->driver = $this->env->getString('STORAGE_DRIVER', 'local', false);
        
        $this->drivers['local'] = $localStorageConfiguration;
        $this->drivers['aws'] = $awsStorageConfiguration;
    }
}

class LocalStorageConfiguration extends AbstractConfig
{
    // ...
    
    public function __construct()
    {
        // ...
    }
}

class AwsStorageConfiguration extends AbstractConfig
{
    // ...
    
    public function __construct()
    {
        // ...
    }
}
```

Now you are able to inject only needed configurations and structure of those configurations are completely transparent. 

```php
<?php

class LocalStorageAdapter
{
    private LocalStorageConfiguration $config;
    
    public function __construct(LocalStorageConfiguration $config)
    {
        $this->config = $config;
    }
    
    public function build()
    {
        $this->config;
    }
}
```

# Documentation

Available methods:

```php
public function getBool(string $key, $default = null, bool $nullable = true): ?bool
```

```php
public function getFloat(string $key, $default = null, bool $nullable = true): ?float
```

```php
public function getInt(string $key, $default = null, bool $nullable = true): ?int
```

```php
public function getList(string $key, $default = [], bool $nullable = false): ?array
```

```php
public function getString(string $key, $default = null, bool $nullable = true): ?string
```

```php
public function getRaw(string $key)
```

🥔

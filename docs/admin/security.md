---
id: security
title: Security
---

This guide describes recommended configurations and practices in order to keep your network secure.

 Enable Production Mode
--------------------------

1. By default, HumHub is shipped in debug mode. After a successful installation, it automatically switches to production mode.
If you previously enabled ``DEBUG`` mode via the ``.env`` file, make sure that ``HUMHUB_DEBUG=0`` before opening your installation to the public.
> If you do not have a ``.env`` file in the project root, you can skip this step.

2. Delete the ``index-test.php`` file in your HumHub root directory if it exists.

Protected Directories
---------------------

Please make sure you followed the directory permissions described in the [Installation Guide](installation.md#file-permissions)!

Limit User Access
-----------------

If you're running a private social network, make sure the user registration has been disabled or the approval system for new users has been enabled.

- Disable user registration: `Administration > Users > Settings > Anonymous users can register`
- Enable user approvals: `Administration > Users > Settings > Require group admin approval after registration`
- Make sure guest access is disabled: `Administration > Users > Settings > Allow limited access for non-authenticated users (guests)`

Password Strength Configuration
-------------------------------

HumHub provides an option for adding of additional validation rules for user password during registration using regular expressions. 
Additional password validation rules can be configured, by changing applications parameters withing the **protected/config/common.php** configuration 

```php
return [
    'modules' => [
        'user' => [
            'passwordStrength' => [
                '/^.{8,}$/' => 'Password needs to be at least 8 characters long.',
                '/^(.*?[A-Z]){2,}.*$/' => 'Password has to contain at least two uppercase letters.',
               	'/^(.*?[a-z]){1,}.*$/' => 'Password has to contain at least one lower case letter.',
               	'/^(.*?[\W]){1,}.*$/' => 'Password has to contain at least one special case letter.',
               	'/^(.*?[0-9]){1,}.*$/' => 'Password has to contain at least one digit.',
            ]
        ]
    ]
];
```

Key should be a valid regular expression, and value - error message.
To localize error message you have to define a new message file with the following path pattern:

`protected/config/messages/<language>/UserModule.custom.php`

Web Security Configuration
---------------------

HumHub comes with a build in web security configuration used to set security headers and csp rules. The default security
configuration can be found at `protected/humhub/config/web.php`.

### Disable Javascript Nonce

Since HumHub 1.15, Javascript CSP Nonce is required and enabled by default. To disable this, please add following lines to your configuration.

**protected/config/web.php:**

```php
return [
    'modules' => [
        'web' => [
            'security' =>  [
                'csp' => [
                    'nonce' => false
                ]
            ]
        ]
    ]
]
```

### Strict CSP Settings

The strictest CSP settings for your installation highly depend on the used features as installed modules, configured oembed provider or
custom iframe pages etc. 

The following example demonstrates a stricter web security model:

**protected/config/web.php:**

```php
return [
    'modules' => [
        'web' => [
            'security' =>  [
                "headers" => [
                    "Strict-Transport-Security" => "max-age=31536000",
                    "X-XSS-Protection" => "1; mode=block",
                    "X-Content-Type-Options" => "nosniff",
                    "X-Frame-Options" => "deny",
                    "Referrer-Policy" => "no-referrer-when-downgrade",
                    "X-Permitted-Cross-Domain-Policies" => "master-only",
                ],
                "csp" => [
                    "nonce" => true,
                    "report-only" => false,
                    "report" => false,
                    "default-src" => [
                        "self" => true
                    ],
                    "img-src" => [
                        "data" => true,
                        "allow" => [
                            "*"
                        ]
                    ],
                    "font-src" => [
                        "self" => true
                    ],
                    "style-src" => [
                        "self" => true,
                        "unsafe-inline" => true
                    ],
                    "object-src" => [],
                    "frame-src" => [
                        "self" => true
                    ],
                    "script-src" => [
                        "self" => true,
                        "unsafe-inline" => true,
                        "unsafe-eval" => false,
                        "report-sample" => true
                    ],
                    "upgrade-insecure-requests" => true
                ]
            ]
        ]
    ]
]
```

There are three main configuration section within your security settings as described in the following:

`headers`:

This part may contain security headers and values as for example:

- [Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
- [X-XSS-Protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)
- [X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)
- [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)
- [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)
- [X-Permitted-Cross-Domain-Policies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)

If you want to add a `Content-Security-Policy` header in the `headers` section of your configuration, remove the `csp` section.

`csp`:

The csp section is used to configure the [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
which manages allowed resources as for example scripts, images and stylesheets. 

Please refer to the following links for more information about the CSP and the configuration format used in HumHub:

- [Content-Security-Policy (MDN web docs)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
- [CSP Builder](https://github.com/paragonie/csp-builder#example)

> Note: the examples shown in the CSP Builder documentation use the JSON format while the HumHub configuration uses a PHP array format.

`csp-report-only`:

This section can be used to define a csp rule, which will only log violations to `Administration > Information > Logging`
rather than blocking the resources on the client. This can be used to test csp rules on your installation.

**CSP Reporting:**

As described above, the `csp-report-only` section of your web security configuration can be used to define csp rules
which are only used for debugging and testing and do not have any effect on the client. The `csp-report-only` can be used along
with the `csp` configuration.

It is also possible to set the `report` setting of your `csp` section to true, this will enable csp violation logging
while enforcing the csp rule.

**CSP Nonce:**

The csp also supports a [nonce](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src) 
settings for your `script-src`. Modern browsers will only execute scripts containing a generated nonce token.

> Note: Some settings as the nonce configuration, may not be supported by some modules. In case you notice modules not working
properly with your security configuration, please contact the module owner or refer to the module description. Also check the 
[Developer Javascript Guide](../develop/javascript-index.md) for assuring nonce support of your custom modules.

> Note: The security rules are cached, you may have to clear the cache in order to update the active rule configuration.

**CSP Guideline:**

This section assembles some guidelines and restrictions regarding custom CSP settings in HumHub.

- The HumHub core currently requires `img-src data:` for page icon and image upload `Administration > Settings > Appearance`
- When using the enterprise edition you should allow `https://www.humhub.org` for `frame-src`
- When noticing any issues with external modules, please inform the module owner.
- When developing custom modules, try to test against the strictest csp rules (see default acceptance test csp rules) and provide
information about csp restrictions in your module description.

Keep HumHub Up-To-Date 
---------------------------------------

As an admin you'll receive notifications about new HumHub releases. We strongly recommend to always update to the latest
stable version if possible.
Check the [update guide](updating.md) for more information about updating your HumHub installation.

Furthermore, you should regularly check the `Administration > Modules > Available Updates` section for module updates. 

We take security very seriously, and we're continuously improving the security features of HumHub. 

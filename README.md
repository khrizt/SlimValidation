# Slim Validation

[![SensioLabsInsight](https://insight.sensiolabs.com/projects/bdf52753-f379-41c6-85cf-d1d1379b4aa7/mini.png)](https://insight.sensiolabs.com/projects/bdf52753-f379-41c6-85cf-d1d1379b4aa7) [![Latest Stable Version](https://poser.pugx.org/awurth/slim-validation/v/stable)](https://packagist.org/packages/awurth/slim-validation) [![Total Downloads](https://poser.pugx.org/awurth/slim-validation/downloads)](https://packagist.org/packages/awurth/slim-validation) [![License](https://poser.pugx.org/awurth/slim-validation/license)](https://packagist.org/packages/awurth/slim-validation)

A validator for the Slim PHP Micro-Framework, using [Respect Validation](https://github.com/Respect/Validation)

## Installation
``` bash
$ composer require awurth/slim-validation
```

## Configuration
You can add the validator to the app container to access it easily through your application
``` php
$container['validator'] = function () {
    return new Awurth\SlimValidation\Validator();
};
```

## Usage
``` php
use Respect\Validation\Validator as V;

// This will return the validator instance
$validator = $container->validation->validate($request, [
    'get_or_post_parameter_name' => V::length(6, 25)->alnum('_')->noWhitespace(),
    // ...
]);

if ($validator->isValid()) {
    // Do something...
} else {
    $errors = $validator->getErrors();
}
```

### Custom messages
You can define messages for a single parameter or global messages for a validation rule, or both.
Individual messages override global messages.

#### Individual messages
``` php
$container->validator->validate($request, [
    'get_or_post_parameter_name' => [
        'rules' => V::length(6, 25)->alnum('_')->noWhitespace(),
        'messages' => [
            'length' => 'Custom message',
            'alnum' => 'Custom message',
            // ...
        ]
    ],
    // ...
]);
```

#### Global messages
``` php
$container->validator->validate($request, [
    'get_or_post_parameter_name' => V::length(6, 25)->alnum('_')->noWhitespace(),
    // ...
], [
    'length' => 'Custom message',
    'alnum' => 'Custom message',
    // ...
]);
```

## Twig extension
To use the extension, you must install twig first
``` bash
$ composer require slim/twig-view
```

### Functions
``` twig
{# Use has_errors() function to know if a form contains errors #}
{{ has_errors() }}

{# Use has_error() function to know if a request parameter is invalid #}
{{ has_error('param') }}

{# Use error() function to get the first error of a parameter #}
{{ error('param') }}

{# Use errors() function to get all errors #}
{{ errors() }}

{# Use errors() function with the name of a parameter to get all errors of a parameter #}
{{ errors('param') }}

{# Use val() function to get the value of a parameter #}
{{ val('param') }}
```

## Example
##### AuthController.php
``` php
public function register(Request $request, Response $response)
{
    if ($request->isPost()) {
        $this->validator->validate($request, [
            'username' => V::length(6, 25)->alnum('_')->noWhitespace(),
            'email' => V::notBlank()->email(),
            'password' => V::length(6, 25),
            'confirm_password' => V::equals($request->getParam('password'))
        ]);
        
        if ($this->validator->isValid()) {
            // Register user in database
            
            return $response->withRedirect('url');
        }
    }
    
    return $this->view->render($response, 'register.twig');
}
```

##### register.twig
``` twig
<form action="url" method="POST">
    <input type="text" name="username" value="{{ val('username') }}">
    {% if has_error('username') %}<span>{{ error('username') }}</span>{% endif %}
    
    <input type="text" name="email" value="{{ val('email') }}">
    {% if has_error('email') %}<span>{{ error('email') }}</span>{% endif %}
    
    <input type="text" name="password">
    {% if has_error('password') %}<span>{{ error('password') }}</span>{% endif %}
    
    <input type="text" name="confirm_password">
    {% if has_error('confirm_password') %}<span>{{ error('confirm_password') }}</span>{% endif %}
</form>
```

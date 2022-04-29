# Laravel Custom Validation Rule


This repository was made to help Laravel developers in making a custom validation rule which you can validate any request's data with any custom condition with.

## Get Started

For example, you're an authenticated admin and you're making your dashboard which have "settings" section where you can change your password at. So you made a form of 3 inputs "Current password", "New password" and "Password confirm". Now, we'll make a `check_password` rule and give it to "Current Password" input to validate if it's a correct password or not.

Make a new `ServiceProvider` by running this artisan command in the terminal:

```bash
php artisan make:provider ValidatorServiceProvider
```
Now, we have to register the new provider we've just made by going to **_config/app.php_**. Scroll down till you find `providers` key and add `ValidatorServiceProvider::class` like below:
```php
    'providers' => [
        # ...
        # ... Other providers
        # ...
        ValidatorServiceProvider::class, // Add this without forgetting to import it with `use` in top of file.
    ]
```

Then, go to **_app/providers/ValidatorServiceProvider.php_** , you'll see method called `boot()` below.

Here we'll configure the new rule `check_password` as below:
```php
  /**
   *
   * $attribute variable expresses the input's name.
   * $value variable expresses the input's value.
   * $parameters variable expresses the parameters you'll pass when you use the rule.
   *
   */
  public function boot()
  {
      $this->app['validator']->extend('check_password', function ($attribute, $value, $parameters) {
          if(!Hash::check($value, auth($parameters[0])->user()->getAuthPassword()))
              return false;
          return true;
      });
  }
```
_Note: It's recommended to name the rule with_ **snake_case** _not_ **camelCase**.

Inside the function, we wrote a condition to check if the provided hashed password is compatible with the one which stored in "admins" table. If it's not, it returns `false` which seems to flash an error in the form. We'll see it now in the usage.

## Usage
In the request class which validates the form, you can so easily give `check_password` rule with passing any guard you want.
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class EditPasswordRequest extends FormRequest
{

    public function authorize()
    {
        return true;
    }

    public function rules()
    {
        return [
            'current_password' => 'required|check_password:admin', // Add the rule for the input.
            'password' => 'required|min:8|confirmed',
            'password_confirmation' => 'required',
        ];
    }

    public function messages()
    {
        return [
            'current_password.required' => 'This field is required.',
            'current_password.check_password' => 'Wrong password.', // Here we are!
            # ...
            # ... Other rules' messages
            # ...
        ];
    }
}
```

=============
CookiesBundle
=============

.. GITHUB_HIDEME .. class:: fa fa-github
.. GITHUB_HIDEME
.. GITHUB_HIDEME `\   \ Fork us on GitHub <https://github.com/ongr-io/CookiesBundle>`_

Cookies bundle provides cookie model abstraction tag.
It can be used as a Symfony service.

Working with cookies
--------------------

How to define a cookie model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ONGR provides cookie model abstraction for working with cookie values in the request and response.

One can define a service:

.. code-block:: yaml

    parameters:
        project.cookie.foo.name: cookie.foo
        project.cookie.foo.defaults: # Defaults section is optional
            http_only: false
            expires_interval: P5DT4H # 5 days and 4 hours

    services:
        project.cookie.foo:
            class: %ongr_cookie.json.class%
            arguments: [ %project.cookie.foo.name% ]
            calls:
                - [setDefaults, [%project.cookie.foo.defaults%]] # Optional
            tags:
                - { name: ongr_cookie.cookie }
            
..

Such injected service allows accessing cookie value. If the value has been modified by your code, it will send new value back to the client browser.

.. code-block:: php

    class CookieController
    {
        use ContainerAwareTrait;
    
        public function updateAction()
        {
            /** @var JsonCookie $cookie */
            $cookie = $this->container->get('project.cookie.foo');
            $cookie->setValue(['bar']);
            // Cookie has been marked as dirty and will be updated in the response.
            $cookie->setExpiresTime(2000000000);
    
            return new JsonResponse();
        }
    }

..

Default values
~~~~~~~~~~~~~~

Possible `setDefaults` keys (default values if unspecified):

- ``domain`` - string (null)

- ``path`` - string ('/')

- ``http_only`` - boolean (true)

- ``secure`` - boolean (false)

- ``expires_time`` - integer (0)

- ``expires_interval`` - `DateInterval <http://php.net/manual/en/dateinterval.construct.php>`_ string (null)

These values are used to initialize the cookie model if cookie does not exist in client's browser.

Model types
~~~~~~~~~~~

Currently, there are these preconfigured classes one can use:

- ``%ongr_cookie.json.class%`` - one can work with it's value as it was a PHP array. In the background, value is encoded and decoded back using JSON format.

- ``%ongr_cookie.generic.class%`` works with plain string data. Other cookie formats can be created by extending this class.

Manually setting cookie
~~~~~~~~~~~~~~~~~~~~~~~

If a cookie with the same name, path and domain is added to the response object, it's value is not overwritten with the changed cookie model data.

Deleting cookie
~~~~~~~~~~~~~~~

To remove a cookie from the client browser, use ``$cookie->setClear(true)``. All other model values will be ignored.

How to perform A B testing
~~~~~~~~~~~~~~~

# Introduction

A/B testing can be implemented using ongr-utils for modifying cookies from outside the ONGR system.

# Configuration

In order to display 2 variations for button color for site visitors, add new user setting and cookie to store that setting:

```yaml
parameters:
    project.cookie.ab_button.name: project_ab_button_color

services:
    project.cookie.ab_button:
        class: %ongr_cookie.cookie.json.class%
        arguments: [ %project.cookie.ab_button.name% ]
        tags:
            - { name: ongr_cookie.cookie }

    ongr_cookie.settings.settings:
        # ...
        ab_button_color:
            name: "Button color"
            category: ab
            description: "Color class for button color"
            cookie: project.cookie.ab_button
            type: [choice, { choices: { red: Red, green: Green } }]
        # ...

    ongr_cookie.settings.categories:
        # ...
        ab:
            name: "A/B Testing"
        # ...
```

More information about cookies in [[How to work with cookies]] page.
# External tools

Use external tool (e.g. Google Analytics, Visual Optimizer, etc.) to set the cookie with name `project_ab_button_color` and value `{"ab_button_color": "red"}` or `"green"` for another visitor group.

*Remember to URL encode the cookie value if needed*.

# Rendering

Use cookie value in TWIG:

```twig
{% set button_color_class = ongr_setting_enabled('foo_setting_2', false) %}
{{ form_widget(form['button_foo'], { 'attr': {'class': button_color_class} }) }}
```

More information in [[Power User]] wiki page.

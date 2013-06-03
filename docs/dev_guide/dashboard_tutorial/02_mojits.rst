============================
2. Mojits: Powering Your App
============================

Introduction
============

Mojits are basic unit of composition and reuse in a Mojito application. 
Imagine that a rectangular square of a Web page being the visual 
representation of a mojit. The word mojit is a compound consisting of 
the stems module and widget, but the mojit is neither. The mojit is built 
on YUI modules, but is not a module, and although can be used in separation 
of other mojits, is not a standalone application like a widget. Regardless 
of how you try to formally understand mojits, you must understand this: 
mojits power your applications.

In this module, we will creating several mojits and then looking at the 
MVC structure underneath the hood. As you We’ll also discuss the difference 
between a mojit definition and a mojit instance and how those are 
manifested in configuration. 

Estimated Time
--------------

15 minutes

What We’ll Cover
----------------

- mojit definitions and instances
- mojit MVC
- ActionContext and ActionContext addons
- views and templates

Final Product
-------------

You’ll see that our application now shows something other than the 
default information. We’ve created new mojits and reworked the 
githubMojit to show some information, which required changing the 
controller, model, and view (templates).

Before Starting
---------------

Review of the Last Module
#########################

In the last module, we discussed the uses of the Mojito 
command-line utility and then learned how to use the basic 
commands to do the following:

- create applications and mojits
- run application and mojit unit tests
- lint code
- start applications
- specify ports and contexts when starting applications.

Setting Up
##########

We’ll be starting where you left off from the last module, 
so you’ll want to make a copy of the application that we made:

``$ cp -r 01_mojito_cli_basics 02_mojits``


Lesson: Mojits and More Mojits
==============================

What is a Mojit?
----------------

Mojits can be thought of as both physical entities represented physically 
by a directory structure and as executable code. The physical files of a 
mojit are what we call the mojit definition and where you most clearly see 
mojit’s MVC structure of the mojit. The executable code, or the instantiation 
of the mojit definition, is called the mojit instance.

We’re going to take a closer look at both the mojit definition and instance, 
how they are created and used.

Mojit Definitions
-----------------

To create the mojit definition, you use the command create that we looked 
at in the previous module. When you run ``mojito create mojit <mojit_name>``, 
Mojito creates a directory structure with boilerplate code for your controller, 
model, view as well as tests and configuration. 

As you can see from the directory structure of the default mojit below, 
the mojits live under the mojits directory and have subdirectories for models, 
tests, and views:

:: 

   testMojit
   ├── assets
   │   └── index.css
   ├── binders
   │   └── index.js
   ├── controller.server.js
   ├── defaults.json
   ├── definition.json
   ├── models
   │   ├── foo.server.js
   │   └── model.server.js
   ├── tests
   │   ├── controller.server-tests.js
   │   └── models
   │       └── foo.server-tests.js
   └── views
       └── index.hb.html


Controllers, Models, and YUI
----------------------------

If you have worked with YUI before, you’ll notice pretty quickly that 
mojit controllers and models are simply custom YUI modules that are 
registered with ``YUI.add``. These skeleton of these custom modules also include 
the requires array that allows you to list dependencies and a namespace:

``controller.server.js``

.. code-block:: javascript

   YUI.add('testMojit', function (Y, NAME) {

     Y.namespace('mojito.controllers')[NAME] = {
     // Code here
     };
   }, '0.0.1', {requires: ['mojito', 'mojito-models-addon', 'testMojitModelFoo']});

``foo.server.js``

.. code-block:: javascript

   YUI.add('testMojitModelFoo', function(Y, NAME) {
    
     Y.namespace('mojito.models')[NAME] = {
       init: function(config) {
         Y.log(config);
         this.config = config;
       }
     };
   }, '0.0.1', {requires: []});

ActionContext Object
--------------------

In mojit controllers, functions in the mojito.controller namespace are 
passed a special object called the Action Context. We’ll be calling it 
the ActionContext object or ac for short.

The Action Context gives you access to important features of the Mojito 
framework. One critical feature is the ability to send data to templates and 
have those templates executed. Mojito also provides a library that can be 
accessed through the ac object through a mechanism called addons.  We’ll 
take a look at done.

For your mojits to render templates, controller functions need to call 
the method ac.done. The done method also allows you to choose the view to 
render and pass meta data, which we will cover in later chapters. If a 
routing path is mapped to an action (controller function) that doesn’t 
call ac.done, your application will hang until it times out.  

ActionContext Addons
--------------------

The Action Context addons provide functionality that lives both on the 
server and client. Each addon provides additional functions through a 
\namespacing object that is appended to the ActionContext object. To use 
addons, function, the addons need to require addons. The default Mojito 
application uses the Models and Assets addon. As our application gets 
more complicated, we’ll be relying on addons to do more work. 

Features
########

The Action Context addons allow you to do the following:

- access assets, such as CSS and JavaScript files
- get configuration information (application.json, routes.json, defaults.json, definition.json)
- get and set cookies
- localize content
- access query and response parameters
- get and set HTTP headers
- create URLs

Syntax
######

Using the ActionContext object ac, you would call a {method} from an {addon} 
with the following syntax: ``ac.{addon}.{method}``

For example, in the application that we will be building, we use the Config 
addon to get the value for the key title: ``ac.config.get('title')``


Requiring Addons
----------------

You require an addon by including an associated string in the requires 
array of your controller. For example, in the controller below, the Config addon 
is required by adding the string ``'mojito-config-addon'`` to the ``requires`` array.

.. code-block:: javascript

   YUI.add('Foo', function(Y, NAME) {
     Y.namespace('mojito.controllers')[NAME] = {
       index: function(ac) {
         var title = ac.config.get(‘title’);
       }
     };
     // Require the addon by adding the param name to the requires array
   }, '0.0.1', {requires: ['mojito', 'mojito-config-addon']});


Controller Methods
------------------

When we say controller methods or functions we are referring to those 
methods in the controller namespace as shown below. 

.. code-block:: javascript

   ...
     Y.namespace('mojito.controllers')[NAME] = {
       index: function(ac) {
         ac.done({ data: { status: “This is an example controller method.” }});
     };
   ...


Views 
-----

The views for Mojito applications are template files. The default templating 
system used by Mojito is Handlebars. You can use other templating systems 
as well, but the for the purpose of this tutorial, we’ll only be covering 
Handlebars.

If you know nothing about Handlebars, we suggest that you read the Handlebars 
documentation. When a controller function calls ac.done with an object as a parameter, 
the object can be passed to the template file. The value of the property or 
key will replace the Handlebars expression.

For example, in the controller function index below, the object ``{ status: “It’s working” }`` 
is passed by default to the template ``index.hb.html``.

.. code-block:: javascript

   ...
     index: function(ac) {
       ac.done({ status: “This is an example controller method.” });
   ...

In the ``index.hb.html`` file below, the Handlebars expression ``{{status}}`` is replaced by the 
string “This is an example controller method.” when the template is rendered.

.. code-block:: html

   <div id="{{mojit_view_id}}">
     <b>{{status}}</b>
   </div>


.. Left of here at 2:36 p.m.

Mojit Configuration Files
-------------------------

Mojits have two files for defining configurations. The file defaults.json 
allows the mojit to have defaults that can be overridden. The file 
definition.json allows the mojit to define key-value pairs that can 
be accessed by the controller. You can also use the settings property 
to specify a context for a runtime environment.

In the ``defaults.json`` file, you list configurations in the config object as shown below. 
These configurations are defaults that will be used unless a mojit instance has 
configurations with the same keys, which we will look at in the next section on mojit instances.

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "config": {
         "gh_mojito”: "https://github.com/yahoo/mojito.git",
         “gh_yui3”: “https://github.com/yui/yui3.git”
       }
      },
      {
        "settings": [ "environment:development" ],
        "config": {
          "gh_mojito”: "https://github.com/yahoo/mojito.git",
          “gh_mojito_remote”: “git@github.com:yahoo/mojito.git”,
          “gh_yui3”: “https://github.com/yui/yui3.git”,
          “gh_yui3_remote”: “git@github.com:yui/yui3.git”
        }
      }
    ]

The configurations in definition.json do not need to be in a ``config`` object. 
You just list key-value pairs:

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "gh_mojito”: "https://github.com/yahoo/mojito.git",
       “gh_yui3”: “https://github.com/yui/yui3.git”
     },
     {
       "settings": [ "environment:development" ],
       "gh_mojito”: "https://github.com/yahoo/mojito.git",
       “gh_mojito_remote”: “git@github.com:yahoo/mojito.git”,
       “gh_yui3”: “https://github.com/yui/yui3.git”,
       “gh_yui3_remote”: “git@github.com:yui/yui3.git”
     }
   ]



Mojit Instances
---------------

We have already seen that Mojito creates anonymous instances of 
mojit definitions by prepending the symbol @ the the mojit name, 
allowing you to execute a mojit action. Generally though, you define a 
mojit instances in configuration, so that the Mojito framework can create 
the instances. The configuration file that is used for defining mojit instances 
and many other application-level configurations is application.json. When you 
run the start command, the Mojito framework parses and loads the application 
configuration, so mojit instances can be dispatched and their actions 
(controller functions) can be executed.

Configuring a Mojit Instance
----------------------------

Mojit instances are configured in the specs object in application.json. 
You create a named object that has a type property that specifies an existing 
mojit definition. In the example below, the mojit instance foo is defined as 
being of type fooMojit. 

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "specs": {
         “foo”: {
           “type”: “fooMojit”
         }
       }
     }
   ]

Mapping Routing Paths to Actions
--------------------------------

Because of the anonymous mojit instances that Mojito creates with a 
mojit definition, your application also gets some default routing 
paths that let you execute mojit actions with a URL. We use the term 
action to differentiate the controller functions of the mojit definition 
from the same functions run by a mojit instance. When you create a mojit, 
as you might have already guessed, you can use the following URL syntax: 
schema to execute mojit actions:  ``http://{domain}:{port}/@{mojit_name}/{action}/``

As with using anonymous instances, you obviously don’t want to use these 
default routes created by Mojito. You instead map routing paths to mojit 
actions in the configuration file routes.json.  The configuration object that 
defines routing information has properties for defining the routing path, HTTP 
methods that are accepted, parameters, and the mojit actions to execute. In the 
example routes.json below, the root object configures the application to execute 
the action index of the mojit foo when an HTTP GET call is made to the path “/”:

.. code-block:: javascript

   [
     {
       "settings": [ "master" ],
       "root": {
         "path": "/",
         "call": "foo.index",
         "verbs": [ "get" ]
       }
     }
   ]

From HTTP Request to Mojit Action
---------------------------------

The diagram below shows the relationship of mojit definition, mojit instance, 
and routing paths. In addition, you can also see the relationship of the application 
within the framework. Notice also that the mojit controller has the function index that 
maps to the action index specified in the routing configuration.




.. tip::  Nulla mattis volutpat justo, et elementum quam condimentum vel. 
          Cras dignissim hendrerit dui, at mollis nisi commodo in. Integer eget 
          sem velit. Sed tempus est quis ligula vulputate vulputate



Recommended Naming Conventions for Mojits
-----------------------------------------

When create mojits (mojit definitions) with the command-line tool, we will be using upper camel case for
the mojit name, such as ``Github``. For mojit instances, we will be using lower case, such as ``github``.
This is the typical convention when defining a class and creating an object, so you can think of
the mojit definition as the class definition and the mojit instance as an instance or object of that class.

Creating the Application
========================

We’re going to extend the application we created in the last module with several 
mojits and then configure mojit instances and routing paths. 

After you have copied the application that you made in the last module (see Setting Up), 
change into the application 02_mojits.

#. Let’s create mojits that will help generate output for the different parts of 
   the HTML page:

   ::

      $ mojito create mojit Body
      $ mojito create mojit Header
      $ mojito create mojit Footer

#. In the mojits directory, you should now see the four mojits we created: ``Github``, 
   ``BodyMojit``, ``HeaderMojit``, and ``FooterMojit``. We’re going to want to create mojit instances 
   that use the mojit definitions. Edit the ``application.json`` so that it is the same as 
   below (feel free to just replace the content of your ``application.json``):

   .. code-block:: javascript

      [
        {
          "settings": [ "master" ],
          "appPort": "8666",
          "specs": {
            "github": {
              "type": "Github",
              "config": {
                "title": "YUI/Mojito Dashboard Application"
              }
            },
            "header": {
              "type": "Header"
            },
            "body": {
              "type": "Body"
            },
            "footer": {
              "type": "Footer"
            }
          }
        },
        {
          "settings": [ "environment:development" ],
          "staticHandling": {
            "forceUpdate": true
          }
        }
      ]

#. Notice that the instance github has a config object. This allows your 
   instance to access the property title, which we’ll look at soon.

   .. code-block:: javascript

      ...  
        "github": {
          "type": "Github",
          "config": {
            "title": "YUI/Mojito Dashboard Application"
          }
        }
      ...

#. With those freshly created instances, we can now define routing paths 
   that execute mojit actions. Let’s create simple routing paths for 
   each of our instances for testing purposes by modifying ``routes.json``
   to look like the following:

   .. code-block:: javascript

      [
        {
          "settings": [ "master" ],
          "root": {
            "verbs": ["get"],
            "path": "/",
            "call": "github.index"
          },
          "header": {
            "verbs":["get"],
            "path": "/header",
            "call": "header.index"
          },
          "body": {
            "verbs": ["get"],
            "path": "/body",
            "call": "body.index"
          },
          "footer": {
            "verbs": ["get"],
            "path": "/footer",
            "call": "footer.index"
          }
        }
      ]

#. We have our instances and our routing paths. Let’s start our 
   application and try hitting the routing paths below. You’ll see the 
   familiar default page for each path, but we’re going to change that next.

   - http://localhost:8666/
   - http://localhost:8666/header
   - http://localhost:8666/body
   - http://localhost:8666/footer

#. We’re going to work a little with the MVC of ``Github``. Let’s first 
   modify the model so that it passes different data to the controller. 
   We’ll get real data in the future, but for now update the method ``getData``
   in your model (``mojits/Github/models/foo.server.js``) so that it’s the same 
   as the following:

   .. code-block:: javascript

      getData: function(callback) {
        callback(null, { watchers: 1, forks: 1 });
      }

#. We’re also going to update the controller so that we’re passing pseudo 
   GitHub data to the template. Open the controller of ``Github``
   (``mojits/Github/controller.server.js``) in an editor and update the 
   object that is passed to ``ac.done`` and the addons required with the following:

   .. code-block:: 

      ...
        ac.done({
          title: ac.config.get('title'),
          github: data
        });
      ...
      {requires: ['mojito', 'mojito-assets-addon', 'mojito-models-addon', 'GithubModelFoo', 'mojito-config-addon']});

#. Because we’ve modified the object that we are passing to the template, 
   we’ll need to modify the template as well. We’re also going to change 
   the HTML in the template, so you can simply replace the contents of the 
   template ``mojits/Github/views/index.hb.html`` with the following:

   .. code-block:: html

      <div id="{{mojit_view_id}}" class="mojit">
        <h4>{{title}}</h4>
        <div class="mymodule">
          <h3>YUI GitHub Stats</h3>
          {{#github}}
            <div>Github watchers: <span>{{watchers}}</span></div>
            <div>Github forks: <span>{{forks}}</span></div>
          {{/github}}
        </div>
      </div>


#. Alright, we’re ready to try out our application. Let’s first test out the 
   routes header, body, and footer. You should see the default Mojito application.

   - http://localhost:8666/body/
   - http://localhost:8666/header/
   - http://localhost:8666/footer/
#. Now for the finale: let’s go to the route to execute our ``Github``, which 
   we modified the model, controller, and view: http://localhost:8666

   You’ll see that model data was passed to the controller and in turn passed to the 
   template, all according to our plan. 

       

Troubleshooting
===============

Route Not Being Found
---------------------

I started the application, but when I go to http://localhost:8666/body, 
I get the following error: Cannot GET /body

It appears that you started Mojito from the wrong location. Try changing 
to the application directory, which in this example is ``02_mojits``, and then run 
mojito start.

Error: listen EADDRINUSE
------------------------

If you start Mojit and get the following error, it means that Mojito is 
already running. You’ll need to cancel that process before you can restart Mojito.

Summary
=======

We covered a lot of content in his module and still missed a lot of points that 
we hope to capture in the upcoming modules. The main focus of the module was 
on mojits, but that is a fairly meaty topic because the mojit is central to 
Mojito applications and one of the main things that sets it apart from 
other frameworks.

- mojit definitions and instances
- mojit MVC
- ActionContext and ActionContext addons
- mojit and application configuration
- templates for views

Q&A
===

How do you...

Test Yourself
=============

Modify the application that we created so that it...

Terms
=====

- mojit definition
- mojit instance
- Action Context
- ac

Source Code
===========

- [app_part{x}](http://github.com/yahoo/mojito/examples/quickstart_guide/app_part{x})

Further Reading
===============

- Handlebars documentation
- Model-view-controller


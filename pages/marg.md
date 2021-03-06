Title: Marg - A Simple Request Router written in PHP
Summary: Marg is a simple request router written in PHP. This article introduces you to [Marg][MARG]'s API and how you can use it.
Date !2012-11-10 10:00
Tags: php, REST, HTTP
Slug: marg
Category: project
Author: Vaidik Kapoor

[Marg][MARG] is a simple HTTP Request Router, written in PHP. It has a very simple API
and is motivated by my understanding and experience of using existing frameworks
like Django, Flask, Drupal and some other pieces of code I have read in the past.
Though I have written this code for my personal use, its free for anyone to use.
The idea behind writing [Marg][MARG] was to have a boilerplate code for myself which I can
use for my own projects.

[Marg][MARG] has a really simple API and it is really easy to use. And one of the reasons
for this is that it does not do everything for you. It does not follow the 
Batteries Included philosophy. All it does for you is that it routes incoming HTTP
requests for you to the right request handlers. You may use functions or classes
to write your request handlers, and then you define routes for each
of those request handlers.

### The Code ###

So lets just dive into some code:

    <?php

    include 'marg/marg.php';

    $routes = array(
        '/' => 'home',
    );

    function home() {
        echo 'Hello World!';
    }

    Marg::run($routes);

    ?>

So that's a really simple example. What we have done here is that we have
defined a new function (controller) called "home". This is basically a function and we want
it to handle our request everytime a client requests the "/" URI. So to do exactly
the same thing, we have defined this function and we have a $routes variable
to tell [Marg][MARG] to route the incoming request to "/" to this request handler. $routes
is an associative array where the key is the URI for which you want to handle and the value
is the name of the request handler or an array (we will see this later) which will handle the
request. Finally, we call the run() method, passing the $routes variable to it as the only 
argument.

Now open the "/" URI in the browser and you will get a nice "Hello World!".

Let's make things a little more complex. Let's handle different kinds of requests.

    <?php

    include 'marg/marg.php';

    $routes = array(
        '/' => 'home',
    );

    function home() {

        echo '<h1>Hello World!</h1>';

        global $request;

        if ($request->verb == 'GET') {
            echo 'This is a GET request.';
        } else {
            header('HTTP/1.1 405 Method Not Allowed');
            echo 'Method Not Allowed';
        }
    }

    Marg::run($routes);

    ?>

In the above example, we have explicitly made sure that our home controller handles
only GET requests. Rather, we want all the non-GET requests to get rejected. The right
way of doing this is by sending the 405 Method Not Allowed response header. Since PHP
was made for the web, you can determine the type of request through many ways. However,
[Marg][MARG] simplifies it for you so that you don't have to take care of this all the time.
[Marg][MARG] provides you with a $request global object that has information about the incoming
request. The "verb" property of the $request object tells you the "HTTP verb" or HTTP
method used for the incoming request. As you can see, we have added a short snippet to
check the method and if it is not GET, then we send the appropriate header and response.

Another way to do this is implement a "raise" function. The way to do that is this:

    <?php

    function home() {
        echo '<h1>Hello World!</h1>';

        global $request;

        if ($request->verb == 'GET') {
            echo 'This is a GET request.';
        } else {
            raise('405');
        }
    }

    function raise_405() {
        header('HTTP/1.1 405 Method Not Allowed');
        echo '<h1>405: Method Not Allowed</h1>';
    }

    ?>

"raise" is used to raise what I call HTTP exceptions. They are not exceptions like exceptions
in PHP or any programming language for that matter. These are essentially functions that you
might want to use to keep your code clean and readable. The way to do this is implement a 
function named in the following format: raise_HTTP_CODE. So to raise a 405 HTTP exception,
all you have to do is call the raise() function with the code as an argument, like so:
raise('405').

raise() function exists for improving code readability and nothing else only as of now.
It does not do much right now, but there is more that I want to do with it in the future -
just haven't found a way to do it yet.

The above example can also be implemented like so:

    <?php

    include 'marg/marg.php';

    $routes = array(
        '/' => array('home', array('GET', 'POST')),
    );

    function home() {
        echo 'Hello World!';
    }

    Marg::run($routes);

    ?>

This way you can inform [Marg][MARG] about which methods your controller will handle and if a request
comes in with an HTTP method not specified in that array, then a 405 is raised.

Alternatively, you may also write the same $routes variable like so:

    <?php

    $routes = array(
        '/' => array(
            'controller' => 'home',
            'methods' => array('GET', 'POST')
        ),
    );

    ?>

The above method is just another explicit way of expressing what we did in the previous example.

We have seen how to use functions to handle your requests. Now, lets take a look at how you'd do
the same thing using classes in PHP.

    <?php

    include 'marg/marg.php';

    $routes = array(
        '/' => array('home', array('GET', 'POST')),
        '/classes' => 'ClassesExample',
    );

    function home() { 
        ...
    }

    class ClassesExample {
        public function get() {
            echo '<h1>This is an example of how to use classes with [Marg][MARG].</h1>';
            echo '<p>This is a GET request.</p>';
        }

        public function post() {
            echo '<h1>This is an example of how to use classes with [Marg][MARG].</h1>';
            echo '<p>This is a POST request.</p>';
        }
    };

    Marg::run($routes);

    ?>

In the above example, we have used a class instead of a function. And our class implements
a get() method and a post() method to handle GET and POST requests coming to "/classes" URI.
That's all we have to do to handle different requests separately. If an incoming request is
not a GET or POST request (say it is a PUT request), then a 405 will be raised.

### setUp and tearDown ###

[Marg][MARG] also allows you to define setUp and tearDown for incoming requests at two different
levels. To define a setUp and a tearDown for all the incoming requests i.e. the global level,
this is what you'd do:

    <?php

    include 'marg/marg.php';

    $routes = array(
        '/' => array('home', array('GET', 'POST')),
    );

    function home() {
        echo 'Hello World!';
    }

    Marg::run($routes);
    Marg::addSetUp(function () {  echo '<html><head><title>Marg Examples</title></head><body>'; });
    Marg::addTearDown(function () { echo '</body></html>'; });

    ?>

Every time you get an incoming request, first the global setUp callback will be called, then the
controller is called and then the global tearDown callback will be called. In this example, we
have used an anonymous function available in PHP 5.3+.

You may also add setUp and tearDown at controller levels like so:

    <?php

    class ClassesExample {
        public function setUp() {
            echo '<h1>This is an example of how to use classes with [Marg][MARG].</h1>';
            echo '<center>';
        }

        public function tearDown() {
            echo '</center>';
        }

        public function get() {
            echo '<p>This is a GET request.</p>';
        }

        public function post() {
            echo '<h1>This is an example of how to use classes with [Marg][MARG].</h1>';
            echo '<p>This is a POST request.</p>';
        }
    };

    ?>

So now when a request comes in, the global setUp is called first, then the setUp at class level
is called, then the class method is called, then the class tearDown is called and finally the
global tearDown is called.

**Please note** that you can use setUp and tearDown at controller levels only with classes and
not with functions.

### Responses ###

You must have noticed that [Marg][MARG] does not provide any way of simplifying how you generate
your responses. Well, there can be a lot of things that I could have done on that front but
that would have been really a lot of opiniated design decisions that I did not want to
enforce onto others and myself as well because I do tend to change how I write my applications
with time and experience. So how you handle your responses depends entirely on you. You may
want to use a templating engine like [Smarty][S] or may be you want to write a custom response
generation engine yourself. Its up to you!

### How and why? ###

While I was interning at [Wingify][W] (the creators of [Visual Website Optimizer][VWO], check
it out in case you don't know what it is 'coz its awesome!), one of the projects I
was working on was development of a REST API for [Visual Website Optimizer][VWO] (VWO).
I hadn't worked on a dedicated REST API before this, mostly because there was 
never a need to do the same. But I'd always think of working on one for a project,
just didn't know that it would be [VWO][VWO]. Anyways, I wanted to get everything right
and so I started to read about API design and best practices. While doing this,
one good thing that happened with me was that I got in the habbit of reading
specs. I read many parts of the HTTP 1.1 spec many times. I read docs of many
web services' APIs like Github's, Facebook's, Google Data Protocol's, and many
more. And, I got to learn a great deal of things, how they are done, why they are
done, etc.

Put together the knowledge and experience I had gathered by using various 
frameworks over the last couple of years and all the information and the new 
things that I had gathered while learning about API design and best practices, I
started working on designing the API spec and then started writing code for the
same. It was an amazing experience. After every addition to the code base, I used
to validate my work with the best practices I had layed down for our use case. And
in the whole process I learned a lot of things.

Sometime back, I was working on a small weekend project and had to write an API for it
that would send responses in JSON. After working on it for a while, I realized my 
approach and code were quite similar to the one I applied while working on the [VWO][VWO]
API project. Therefore, I decided to make use of all the things I learned while working
at [Wingify][W] and write my own boilerplate code for my own use. So this became my new weekend
project which resulted in [Marg][MARG].

I thank [Wingify][W] here for giving me the opportunity to work on that project. I got the 
opportunity to learn a great deal through the project I was asked to work on!

### Feedback ###

[Marg][MARG] was more of a weekend project. I have added code to it but haven't been able to really
devote time to it. I am sure it has some glitches. I'd be really happy to have some feedback.
So, please take out some time and let me know how you feel about it.

[Marg]: http://github.com/vaidikkp/marg
[W]: https://wingify.com
[VWO]: http://visualwebsiteoptimizer.com
[S]: http://www.smarty.net/

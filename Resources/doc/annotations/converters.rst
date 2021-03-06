@ParamConverter
===============

Usage
-----

The ``@ParamConverter`` annotation calls *converters* to convert request
parameters to objects. These objects are stored as request attributes and so
they can be injected as controller method arguments::

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;

    /**
     * @Route("/blog/{id}")
     * @ParamConverter("post", class="SensioBlogBundle:Post")
     */
    public function showAction(Post $post)
    {
    }

Several things happens under the hood:

* The converter tries to get a ``SensioBlogBundle:Post`` object from the
  request attributes (request attributes comes from route placeholders -- here
  ``id``);

* If no ``Post`` object is found, a ``404`` Response is generated;

* If a ``Post`` object is found, a new ``post`` request attribute is defined
  (accessible via ``$request->attributes->get('post')``);

* As for any other request attribute, it is automatically injected in the
  controller when present in the method signature.

If you use type hinting as in the example above, you can even omit the
``@ParamConverter`` annotation altogether::

    // automatic with method signature
    public function showAction(Post $post)
    {
    }

To detect which converter is run on a parameter the following process is run:

* If an explicit converter choice was made with
  ``@ParamConverter(converter="name")`` the converter with the given name is
  chosen.
* Otherwise all registered parameter converters are iterated by priority.
  The ``supports()`` method is invoked to check if a param converter can
  convert the request into the required parameter. If it returns ``true``
  the param converter is invoked.

Built-in Converters
-------------------

<<<<<<< HEAD
The bundle has two built-in converter, the Doctrine one and a generic object
=======
The bundle has two built-in converter, the Doctrine one and a DateTime
>>>>>>> DateTimeConverter
converter.

Doctrine Converter
~~~~~~~~~~~~~~~~~~

<<<<<<< HEAD
<<<<<<< HEAD
=======
Converter Name: ``doctrine.orm``

>>>>>>> sensio/master
The Doctrine Converter attempts to convert request attributes to Doctrine
entities fetched from the database. Two different approaches are possible:

- Fetch object by primary key
- Fetch object by one or several fields which contain unique values in the
  database.

The following algorithm determines which operation will be performed.

- If an ``{id}`` parameter is present in the route, find object by primary key.
- If an option ``'id'`` is configured and matches route parameters, find object by primary key.
- If the previous rules do not apply, attempt to find one entity by matching
  route parameters to entity fields. You can control this process by
  configuring ``exclude`` parameters or a attribute to field name ``mapping``.
<<<<<<< HEAD
=======
Converter Name: ``doctrine.orm``
>>>>>>> origin/NamedParamConverters
=======
>>>>>>> sensio/master

By default, the Doctrine converter uses the *default* entity manager. This can
be configured with the ``entity_manager`` option::

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;

    /**
     * @Route("/blog/{id}")
     * @ParamConverter("post", class="SensioBlogBundle:Post", options={"entity_manager" = "foo"})
     */
    public function showAction(Post $post)
    {
    }

If the placeholder has not the same name as the primary key, pass the ``id``
option::

    /**
     * @Route("/blog/{post_id}")
     * @ParamConverter("post", class="SensioBlogBundle:Post", options={"id" = "post_id"})
     */
    public function showAction(Post $post)
    {
    }

This also allows you to have multiple converters in one action::

    /**
     * @Route("/blog/{id}/comments/{comment_id}")
     * @ParamConverter("comment", class="SensioBlogBundle:Comment", options={"id" = "comment_id"})
     */
    public function showAction(Post $post, Comment $comment)
    {
    }

In the example above, the post parameter is handled automatically, but the comment is 
configured with the annotation since they can not both follow the default convention.

<<<<<<< HEAD
<<<<<<< HEAD
Object Converter
----------------

If the Doctrine Converter (or any other Persistent Object Converter) could not
convert an object, you can use the generic object converter to map request data
to an object.

- Only data from the request query is used to construct the target object.
  To work with POST data, you should use the form framework.
- If the data is an array, the keys are matched against parameter variable names in the
  target object constructor.
- If the data is a scalar value it is assumed to be a one-parameter
  constructor.
- The request data is only matched against the constructor of the target
  object to allow the developer to have full control over the accepted user
  input.
- The construction is recursive and allows to assemble object graphs. DateTime
  is handled as special case.

Example::

    /**
     * @Route("/blog")
     */
    public function listAction(PostCriteria $criteria)
    {
    }

    class PostCriteria
    {
        private $page;
        private $count;

        public function __construct($page = 1, $count = 20)
        {
            $this->page = $page;
            $this->count = $count;
        }
    }

Example requests for this action could be:

    curl http://example.com/blog
    curl http://example.com/blog?criteria[page]=4&criteria[count]=50

.. note::

    For security reasons the object converter has to run AFTER persistent
    object parameter converters such as the DoctrineParam Converter. Otherwise
    attackers could inject objects in your action that would normally be
    persistent objects and not objects from user input.

=======
=======
If you want to match an entity using multiple fields use ``mapping``::

    /**
     * @Route("/blog/{date}/{slug}/comments/{comment_slug}")
     * @ParamConverter("post", options={"mapping": {"date": "date", "slug": "slug"})
     * @ParamConverter("comment", options={"mapping": {"comment_slug": "slug"})
     */
    public function showAction(Post $post, Comment $comment)
    {
    }

If you are matching an entity using several fields, but you want to exclude a
route parameter from being part of the criteria::

    /**
     * @Route("/blog/{date}/{slug}")
     * @ParamConverter("post", options={"exclude": ["date"]})
     */
    public function showAction(Post $post, \DateTime $date)
    {
    }

>>>>>>> sensio/master
DateTime Converter
~~~~~~~~~~~~~~~~~~

Converter Name: ``datetime``

The datetime converter converts any route or request attribute into a datetime
instance::

    /**
     * @Route("/blog/archive/{start}/{end}")
     */
    public function archiveAction(DateTime $start, DateTime $end)
    {
    }

By default any date format that can be parsed by the ``DateTime`` constructor
is accepted. You can be stricter with input given through the options::

    /**
     * @Route("/blog/archive/{start}/{end}")
     * @ParamConverter("start", options={"format": "Y-m-d"})
     * @ParamConverter("end", options={"format": "Y-m-d"})
     */
    public function archiveAction(DateTime $start, DateTime $end)
    {
    }

>>>>>>> DateTimeConverter
Creating a Converter
--------------------

All converters must implement the
:class:`Sensio\\Bundle\\FrameworkExtraBundle\\Request\\ParamConverter\\ParamConverterInterface`::

    namespace Sensio\Bundle\FrameworkExtraBundle\Request\ParamConverter;

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\ConfigurationInterface;
    use Symfony\Component\HttpFoundation\Request;

    interface ParamConverterInterface
    {
        function apply(Request $request, ConfigurationInterface $configuration);

        function supports(ConfigurationInterface $configuration);
    }

The ``supports()`` method must return ``true`` when it is able to convert the
given configuration (a ``ParamConverter`` instance).

The ``ParamConverter`` instance has three information about the annotation:

* ``name``: The attribute name;
* ``class``: The attribute class name (can be any string representing a class
  name);
* ``options``: An array of options

The ``apply()`` method is called whenever a configuration is supported. Based
on the request attributes, it should set an attribute named
``$configuration->getName()``, which stores an object of class
``$configuration->getClass()``.

To register your converter service you must add a tag to your service

.. configuration-block::

    .. code-block:: xml

        <service id="my_converter" class="MyBundle/Request/ParamConverter/MyConverter">
            <tag name="request.param_converter" priority="-2" name="my_converter" />
        </service>

You can register a converter by priority, by name or both. If you don't
specifiy a priority or name the converter will be added to the converter stack
with a priority of `0`. To explicitly disable the registration by priority you
have to set `priority="false"` in your tag definition.

.. tip::

   Use the ``DoctrineParamConverter`` class as a template for your own converters.


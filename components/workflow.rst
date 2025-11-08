The Workflow Component
======================

    The Workflow component provides tools for managing a workflow or finite
    state machine.

Installation
------------

.. code-block:: terminal

    $ composer require symfony/workflow

.. include:: /components/require_autoload.rst.inc

Creating a Workflow
-------------------

The workflow component provides an object-oriented way to define a process
or the life cycle that an object undergoes. Each step or stage in the
process is called a **place**, and each action that connects these places
is called a **transition**. The options available for transitions depend
on the workflow type; for more details, 
:doc:`see article </workflow/workflow-and-state-machine>` 

.. image:: /_images/components/workflow/states_transitions.png
    :alt: An example state diagram for a workflow, showing transitions and places.

A set of places and transitions creates a **definition**. A workflow needs
a definition and a way to write the states to the objects (i.e. an
instance of a :class:`Symfony\\Component\\Workflow\\MarkingStore\\MarkingStoreInterface`).

Consider the following example for a simple blog post process. A post can have these places:
``draft``, ``reviewed``, ``rejected``, ``published``. You could define the workflow as
follows::

    use Symfony\Component\Workflow\DefinitionBuilder;
    use Symfony\Component\Workflow\MarkingStore\MethodMarkingStore;
    use Symfony\Component\Workflow\Transition;
    use Symfony\Component\Workflow\Workflow;

    $definitionBuilder = new DefinitionBuilder();
    $definition = $definitionBuilder->addPlaces(['draft', 'reviewed', 'rejected', 'published'])
        // Transitions are defined with a unique name, an origin place and a destination place
        ->addTransition(new Transition('review', 'draft', 'reviewed'))
        ->addTransition(new Transition('publish', 'reviewed', 'published'))
        ->addTransition(new Transition('reject', 'reviewed', 'rejected'))
        ->build()
    ;

    $singleState = true; // true if the subject can be in only one state at a given time
    $property = 'currentState'; // subject property name where the state is stored
    $marking = new MethodMarkingStore($singleState, $property);
    $workflow = new Workflow($definition, $marking);

The ``Workflow`` can now help you to decide what transitions (actions) are allowed
on a blog post depending on what place (state) it is in. This will keep your domain
logic in one place and not spread all over your application.

Transitions can have weight defined that define how many time the transition
is expected to be applied in the workflow. This option is available only for
generic workflows using multiple states.

.. versionadded:: 7.4

    The support for ``weight`` option was introduced in Symfony 7.4.

Usage
-----

Here's an example of using the workflow defined above::

    // ...
    // Consider that $blogPost is in place "draft" by default
    $blogPost = new BlogPost();

    $workflow->can($blogPost, 'publish'); // False
    $workflow->can($blogPost, 'review'); // True

    $workflow->apply($blogPost, 'review'); // $blogPost is now in place "reviewed"

    $workflow->can($blogPost, 'publish'); // True
    $workflow->getEnabledTransitions($blogPost); // $blogPost can perform transition "publish" or "reject"

Initialization
--------------

If the marking property of your object is ``null`` and you want to set it with the
``initial_marking`` from the configuration, you can call the ``getMarking()``
method to initialize the object property::

    // ...
    $blogPost = new BlogPost();

    // initiate workflow
    $workflow->getMarking($blogPost);

Learn more
----------

Read more about the usage of the :doc:`Workflow component </workflow>` inside a Symfony application.

.. toctree::
    :maxdepth: 1
    :glob:

    /workflow/*

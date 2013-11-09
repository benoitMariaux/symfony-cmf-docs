.. index::
    single: PHPCR-ODM, Creating a New Project

Create a New Project with PHPCR-ODM
===================================

This article will show you how to create a new Symfony project from the
`Symfony Standard Edition`_ using PHPCR-ODM instead of (or in addition to) the
`Doctrine ORM`_.

For more information on the PHPCR-ODM see the
:doc:`../book/database_layer` article.

General Instructions using Jackalope Doctrine DBAL
--------------------------------------------------

The `Jackalope`_ Doctrine DBAL backend will use `Doctrine DBAL`_ to store the
content repository.

@TODO: Get Composer / Add prerequisite

**Step 1**: Create a new Symfony project with composer based on the standard edition:

.. code-block:: bash

    $ php composer.phar create-project symfony/framework-standard-edition path/

**Step 2**: Add the required packages to ``composer.json``:

.. code-block:: javascript

    {
        ...
        "require": {
            ...
            "doctrine/phpcr-bundle": "1.0.0",
            "doctrine/doctrine-bundle": "1.2.*",
            "doctrine/phpcr-odm": "1.0.*",
            "jackalope/jackalope-doctrine-dbal": "1.0.0"
        }
    }

**Step 3**: (*optional*) Remove the Doctrine ORM:

Remove the ``doctrine\orm`` package from ``composer.json``.

@TODO: Remove orm section from configuration (failing to do so will result in
crash)::

    orm:
        auto_generate_proxy_classes: %kernel.debug%
        auto_mapping: true

**Step 4**: Add the DoctrinePHPCRBundle to the AppKernel::

    // app/AppKernel.php

    // ...
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
                // ...
                new Doctrine\Bundle\PHPCRBundle\DoctrinePHPCRBundle(),
            );

            // ...
        }
    }

**Step 5**: (*optional*) Register the PHPCR-ODM annotations in ``app/autoload.php``::

    // app/autoload.php

    // ...
    AnnotationRegistry::registerFile(
        __DIR__.'/../vendor/doctrine/phpcr-odm/lib/Doctrine/ODM/PHPCR/Mapping/Annotations/DoctrineAnnotations.php'
    );

**Step 6**: Modify ``parameters.yml.dist``, adding the required PHPCR-ODM settings:

.. code-block:: yaml

    # app/config/parameters.yml.dist
    parameters:
        # ...
        phpcr_backend:
            type: doctrinedbal
            connection: default
        phpcr_workspace: default
        phpcr_user: admin
        phpcr_pass: admin 

.. note::

    The actual database backend (MySQL, sqlite3, Postgres etc) is handled by
    Doctrine DBAL.

@TODO: Why are you modifying "dist" ?

**Step 7**: Add the Doctrine PHPCR configuration to the main application configuration:

.. configuration-block::

    .. code-block:: yaml

        # ...
        doctrine_phpcr:
           # configure the PHPCR session
           session:
               backend: %phpcr_backend%
               workspace: %phpcr_workspace%
               username: %phpcr_user%
               password: %phpcr_pass%
           # enable the ODM layer
           odm:
               auto_mapping: true
               auto_generate_proxy_classes: %kernel.debug%   

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services">
            <config xmlns="http://example.org/schema/dic/doctrine_phpcr">
                <session backend="%phpcr_backend%"
                    workspace="%phpcr_workspace%"
                    username="%phpcr_user%"
                    password="%phpcr_pass%"
                />

                <odm auto-mapping="true"
                    auto-generate-proxy-classes="%kernel.debug%"
                />
            </config>
        </container>

    .. code-block:: php

        $container->loadFromExtension('doctrine_phpcr', array(
            'session' => array(
                'backend' => '%phpcr_backend%',
                'workspace' => '%phpcr_workspace%',
                'username' => '%phpcr_username%',
                'password' => '%phpcr_password%',
            ),
            'odm' => array(
                'auto_mapping' => true,
                'auto_generate_proxy_classes' => '%kernel.debug%',
            ),
        ));

@TODO: Composer update

Alternative Backend: Apache Jackrabbit
--------------------------------------

`Apache Jackrabbit`_ is a mature Java based content repository which can be used
as an alternative to the Jackalope Doctrine DBAL backend.

The instructions are the same as for Doctrine DBAL with the following
differences:

**Step 2**: Include ``jackalope/jackalope-jackrabbit`` instead of
  ``jackalope/jackalope-doctrine-dbal``.

Install and Run the Jackrabbit Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Download Jackrabbit in whatever way you prefer (for example using ``wget``):

.. code-block:: bash

    $ wget http://www.apache.org/dyn/closer.cgi/jackrabbit/2.4.5/jackrabbit-standalone-2.4.5.jar

Start the Jackrabbit server:

.. code-block:: bash

    $ java -jar jackrabbit

This will create a directory called ``jackrabbit`` in the current working
directory which will contain the data of the content repository.

.. _`Symfony Standard Edition`: https://github.com/symfony/symfony-standard
.. _`Doctrine ORM`: https://github.com/doctrine/doctrine2
.. _`Apache Jackrabbit`: https://jackrabbit.apache.org
.. _`Jackalope`: https://github.com/jackalope/jackalope
.. _`Doctrine DBAL`: https://github.com/doctrine/dbal

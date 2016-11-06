# ACL extension for Sonata Admin (Symfony 3 compatible)

This bundle provides ACL list filtering for [SonataAdminBundle](https://github.com/sonata-project/SonataAdminBundle).
When enabled, **list and dashboard** screen only display data the logged in user has right to view.

This bundle is **Symfony 3 compatible** and is a good complementary of the SonataAdminBundle [ACL editor](http://sonata-project.org/bundles/admin/master/doc/reference/security.html#acl-editor).



[![SensioLabsInsight](https://insight.sensiolabs.com/projects/d7d70442-b52c-4072-8e03-45e6a47e1ca2/mini.png)](https://insight.sensiolabs.com/projects/d7d70442-b52c-4072-8e03-45e6a47e1ca2)

## Install

Be sure that SonataAdminBundle is working and has [ACL enabled](http://sonata-project.org/bundles/admin/master/doc/reference/security.html#acl-and-friendsofsymfony-userbundle).

Install this bundle using composer:

```
// composer.json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/Carteni/CoopTilleulsAclSonataAdminExtensionBundle"
        }
    ],
    "require": {
        "tilleuls/acl-sonata-admin-extension-bundle": "dev-master"
    }
}

$ composer update
```

Register the bundle in your AppKernel:

```php
// app/AppKernel.php

public function registerBundles()
{
    return array(
        // ...
        new CoopTilleuls\Bundle\AclSonataAdminExtensionBundle\CoopTilleulsAclSonataAdminExtensionBundle(),
        // ...
    );
}
```

## Enable

This extension is automatically enabled for all admins.

## How To

**#1. Create a new EMAIL (custom) ACL permission.**

```php
// app/config/security.php
security:
    acl:
        connection: default

    access_decision_manager:
            strategy: unanimous

    parameters:
        security.acl.permission.map.class: AppBundle\Security\Acl\Permission\PermissionMap
        sonata.admin.security.mask.builder.class: AppBundle\Security\Acl\Permission\MaskBuilder

// src/AppBundle/Security/Acl/Permission/PermissionMap.php
namespace AppBundle\Security\Acl\Permission;

use Symfony\Component\Security\Acl\Permission\PermissionMapInterface;

/**
 * Class PermissionMap
 * @package AppBundle\Security\Acl\Permission
 */
class PermissionMap implements PermissionMapInterface
{
    const PERMISSION_VIEW       = 'VIEW';
    const PERMISSION_EDIT       = 'EDIT';
    const PERMISSION_CREATE     = 'CREATE';
    const PERMISSION_DELETE     = 'DELETE';
    const PERMISSION_UNDELETE   = 'UNDELETE';
    const PERMISSION_LIST       = 'LIST';

    const PERMISSION_EMAIL      = 'EMAIL';

    const PERMISSION_EXPORT     = 'EXPORT';
    const PERMISSION_OPERATOR   = 'OPERATOR';
    const PERMISSION_MASTER     = 'MASTER';
    const PERMISSION_OWNER      = 'OWNER';

    /**
     * Map each permission to the permissions it should grant access for
     * fe. grant access for the view permission if the user has the edit permission.
     *
     * @var array
     */
    private $map = array(

      self::PERMISSION_VIEW => array(
        MaskBuilder::MASK_VIEW,
        MaskBuilder::MASK_LIST,
        MaskBuilder::MASK_EDIT,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_EDIT => array(
        MaskBuilder::MASK_EDIT,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_CREATE => array(
        MaskBuilder::MASK_CREATE,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_DELETE => array(
        MaskBuilder::MASK_DELETE,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_UNDELETE => array(
        MaskBuilder::MASK_UNDELETE,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_LIST => array(
        MaskBuilder::MASK_LIST,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_EMAIL => array(
        MaskBuilder::MASK_EMAIL,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_EXPORT => array(
        MaskBuilder::MASK_EXPORT,
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_OPERATOR => array(
        MaskBuilder::MASK_OPERATOR,
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_MASTER => array(
        MaskBuilder::MASK_MASTER,
        MaskBuilder::MASK_OWNER,
      ),

      self::PERMISSION_OWNER => array(
        MaskBuilder::MASK_OWNER,
      ),
    );

    /**
     * {@inheritdoc}
     */
    public function getMasks($permission, $object)
    {
        if (!isset($this->map[$permission])) {
            return;
        }

        return $this->map[$permission];
    }

    /**
     * {@inheritdoc}
     */
    public function contains($permission)
    {
        return isset($this->map[$permission]);
    }
}


// src/AppBundle/Security/Acl/Permission/MaskBuilder.php
namespace AppBundle\Security\Acl\Permission;

/**
 * Class MaskBuilder
 * @package AppBundle\Security\Acl\Permission
 */
class MaskBuilder extends \Sonata\AdminBundle\Security\Acl\Permission\MaskBuilder
{
    const MASK_EMAIL = 256;       // 1 << 8

    const CODE_EMAIL = 'E';
}

// app/config/config.yml
sonata_admin:
    ...
    security:
        handler: sonata.admin.security.handler.acl

    information:
        GUEST:    [VIEW, LIST]
        STAFF:    [EDIT, LIST, EMAIL, CREATE]
        EDITOR:   [OPERATOR, EXPORT]
        ADMIN:    [MASTER]

        # permissions *not* related to an object instance
        admin_permissions: [CREATE, LIST, EMAIL, DELETE, UNDELETE, EXPORT, OPERATOR, MASTER]

        # permission related to the objects
        object_permissions: [VIEW, EDIT, DELETE, UNDELETE, OPERATOR, MASTER, OWNER]
```

## TODO

* Test with other DBMSs than MySQL
* Write tests

## Credits

Created by [Kévin Dunglas](http://dunglas.fr) for [Les-Tilleuls.coop](http://les-tilleuls.coop).

Forked and improved by [Francesco Cartenì](http://www.multimediaexperiencestudio.it)


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/coopTilleuls/cooptilleulsaclsonataadminextensionbundle/trend.png)](https://bitdeli.com/free "Bitdeli Badge")


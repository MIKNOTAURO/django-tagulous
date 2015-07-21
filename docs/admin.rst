.. _admin:

Admin
=====

Tag fields in ModelAdmin
------------------------

To support TagField and SingleTagField fields in the admin, you need to
register the Model and ModelAdmin using Tagulous's `register()` function,
instead of the standard one::

    import tagulous
    class MyAdmin(admin.ModelAdmin):
        list_display = ['name', 'tags']
    tagulous.admin.register(MyModel, MyAdmin)

This will make a few changes to ``MyAdmin`` to add tag field support (detailed
below), and then register it with the default admin site using the standard
``site.register()`` call.

As with the normal registration call, the admin class is optional::

    tagulous.admin.register(myModel)

You can also pass a custom admin site into the `register()` function::

    # These two lines are equivalent:
    tagulous.admin.register(myModel, MyAdmin)
    tagulous.admin.register(myModel, MyAdmin, site=admin.site)

The changes Tagulous's ``register()`` function makes to the ``ModelAdmin`` are:

* Changes your ``ModelAdmin`` to subclass ``TaggedAdmin``
* Checks ``list_display`` for any tag fields, and adds functions to the
  ``ModelAdmin`` to display the tag string (unless an attribute with that name
  already exists)

Note:

* You can only provide the Tagulous ``register()`` function with one model.
* The admin class will be modified; bear that in mind if registering it with
  multiple admin sites. In that case, you may want to enhance the class
  manually, as described below.


Manually enhancing your ModelAdmin
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``tagulous.admin.register`` function is the short way to enhance your admin
classes. If for some reason you can't use it (eg another library which has its
own ``register`` function, or you're registering it with more than one admin
site), you can do what it does manually:

1. Change your admin class to subclass ``tagulous.admin.TaggedModelAdmin``.

   This disables Django's green button to add a related field, which is
   incompatible with Tagulous.

2. Call ``tagulous.admin.enhance(model_class, admin_class)``.
   
   This finds the tag fields on the model class, and adds support for them to
  ``list_display``.

3. Register the admin class as normal

For example::

    import tagulous
    class MyAdmin(tagulous.admin.TaggedModelAdmin):
        list_display = ['name', 'tags']
    tagulous.admin.enhance(MyModel, MyAdmin)
    admin.site.register(MyModel, MyAdmin)


Autocomplete settings
---------------------

The admin site can use different autocomplete settings to the public site by
changing the settings ``TAGULOUS_ADMIN_AUTOCOMPLETE_JS`` and
``TAGULOUS_ADMIN_AUTOCOMPLETE_CSS``. You may want to do this to avoid jQuery
being loaded more than once, for example - assuming the version in Django's
admin site is compatible with the autocomplete library of your choice.

See `Settings`_ for more information


Managing the tag model
----------------------

You can also register a ModelAdmin to manipulate the tag table directly.
Tagulous has a helper function to do this for you::

    tagulous.admin.tag_model(MyModel.tags.tag_model)

The example is for an auto-generated tag model, but it could equally be a
custom tag model.

If you have a custom model and want to extend the admin class for extra fields
on your custom model, you can subclass the ``TagModelAdmin`` class to get the
extra tag management functionality::

    class MyModelTagsAdmin(tagulous.admin.TagModelAdmin):
        list_display = ['name', 'count', 'protected', 'my_extra_field']
    admin.site.register(MyModel.tags.tag_model, MyModelTagsAdmin)

Remember that the relationship between your entries and tags are standard
``ForeignKey`` or ``ManyToMany`` relationships, so deletion propagation will
work as it would normally.
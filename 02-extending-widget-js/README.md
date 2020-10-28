# Extending JavaScript for widgets

*This tutorial shows you how to safely extend the JavaScript functionality of a widget in a way that survives October CMS updates.*

Due to the class-less nature of JavaScript, extending the internal front-end functionality of widgets can be difficult. Most widgets in October CMS are kept inside enclosed functions, preventing them from being modified externally or modifying core functionality due to the limited scope of access. 

With this tutorial, I will demonstrate a way to retrieve a widget's JavaScript object to modify or augment the core functionality that emulates PHP class extension capabilities, so that you affect the core functionality as minimally as possible and allow your modifications to work even if the widget's functionality is updated in future October CMS versions.

## Introduction

For this particular tutorial, we'll make an adjustment to the Dashboard widgets' default functionality that shows a confirmation message when removing a widget, prompting the user to confirm the removal. Instead, we'll make it so that there's no confirmation message when you remove a widget - the widget is simply removed. We will do this in the context of a custom October CMS plugin.

The dashboard widgets are contained and handled by the `ReportContainer` widget, which can be found in `/modules/backend/widgets/ReportContainer.php`. The particular JavaScript code that is fired when you remove a widget is this block of code:

https://github.com/octobercms/october/blob/7b8fecaa51cabf059e39536f41c68d03ebb9a901/modules/backend/widgets/reportcontainer/assets/js/reportcontainer.js#L82-L95

```js
this.$el.on('click', '.content > button.close', function() {
    var $btn = $(this)
    $.oc.confirm($.oc.lang.get('alert.widget_remove_confirm'), function() {
        self.$form.request(self.alias + '::onRemoveWidget', {
            data: {
                'alias': $('[data-widget-alias]', $btn.closest('div.content')).val()
            }
        })

        $btn.closest('li').remove()
        self.redraw()
        self.setSortOrders()
    })
})
```

Note that this is contained within a jQuery function block, which defines the `ReportContainer` JavaScript object and functionality, and registers it as a jQuery plugin, so that an element with a `data-control` attribute of `report-container` can become a report container. Due to this being inside a function block, the scope is limited - all variables defined inside this block are limited to this function block only, so you cannot directly write to the `ReportContainer` variable to override or augment the functionality.

However, by exploiting the global jQuery `$fn` object and extracting the `ReportContainer` object from within the jQuery plugin architecture, we can extend and/or overwrite the functionality and define this as the report container functionality. To do this, we'll need to create and load a second JavaScript file that will act as our extension.

## Create the extension

To start, we'll need to create and register our plugin. If you do not know how to do this, it is best to [follow the instructions](https://octobercms.com/docs/plugin/registration) on the October CMS docs to get started.

Once your plugin is created and enabled, we'll create our JavaScript asset. Let's store it inside the plugin folder as `assets/js/reportcontainer.js`. We'll then need to copy over some code from the original widget to act as the "base" of the extension.

```js
+function ($) { "use strict";
    var ReportContainer = function(element, options) {
        this.options = options
        this.$el = $(element)
        this.$form = this.$el.closest('form')
        this.$toolbar = $('[data-container-toolbar]', this.$form)
        this.alias = $('[data-container-alias]', this.$form).val()

        this.init();
    }

    ReportContainer.DEFAULTS = {
        breakpoint: 768,
        columns: 12
    }

    ReportContainer.prototype = Object.create($.fn.reportContainer.Constructor.prototype)

    var old = $.fn.reportContainer

    $.fn.reportContainer = function(option) {
        return this.each(function() {
            var $this   = $(this)
            var data    = $this.data('oc.reportContainer')
            var options = $.extend({}, ReportContainer.DEFAULTS, $this.data(), typeof option == 'object' && option)
            if (!data) $this.data('oc.reportContainer', (data = new ReportContainer(this, options)))
            if (typeof option == 'string') data[option].call($this)
        })
    }

    $.fn.reportContainer.Constructor = ReportContainer

    $.fn.reportContainer.noConflict = function() {
        $.fn.reportContainer = old
        return this
    }

    $(document).render(function() {
        $('[data-control="report-container"]').reportContainer()
    })
}(window.jQuery);
```

The main line in which the magic happens is this line:

```js
ReportContainer.prototype = Object.create($.fn.reportContainer.Constructor.prototype)
```

This line extracts the widget's prototype from the `reportContainer` jQuery plugin - our original report container widget - and copies it into our own extension's prototype. At this point, our extension will work exactly the same as the original

# Extending JavaScript for widgets

*This tutorial shows you how to safely extend the JavaScript functionality of a widget in a way that survives October CMS updates.*

Due to the class-less nature of JavaScript, with most functionality being stored as objects, extending the internal front-end functionality of widgets can be 
difficult. Most widgets in October CMS are kept inside enclosed functions, preventing them from being modified externally or modifying core functionality due to
the limited scope of access. With this tutorial, I will demonstrate a way to retrieve a widget's object to modify or augment the core functionality that emulates
PHP class extension capabilities, so that you affect the core functionality as minimally as possible and allow your modifications to work even if the widget's
functionality is updated in future October CMS versions.

## Introduction

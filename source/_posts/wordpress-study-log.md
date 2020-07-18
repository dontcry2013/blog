---
title: wordpress study log
date: 2020-06-30 22:44:19
categories:
tags:
---
# XAMPP

``` bash
# check default username and password
/Applications/XAMPP/xamppfiles/xampp security
# get right privileges to install & uninstall plugins
chown -R daemon:daemon /Applications/XAMPP/htdocs/wp
```
<!--more-->

# MySQL

For MySQL 8.x, should use mysql_native_password.

``` sql
ALTER USER 'username'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

---
Date 2 JUL

## Basic Settings

* Permalink
* Install theme
* customize theme
  * Header Image
  * Menu Location
  * Widget
* Plugins
  * WPForms
  * Yoast SEO
  * Insert Headers and Footers

## Difference of platforms
* Wix
* Weebly
* Shopify
* Wordpress.com

wp classic editor

---
## vantage theme

https://www.youtube.com/watch?v=eYF23mCGiDw

* lightweight social icon
  * add social iocns to footer
* wp video lightbox
  * to enable clickable video button on banner
* meta slider
* front page control
  * settings -> reading ->a static page -> edit front page, posts page
* [iconmakr](https://logomakr.com/)
* [pixlr](https://pixlr.com/)
* \<a href="" rel="wp-video-lightbox"> to show a video by click
* © 2020 Zach 
* remove the theme copyright text in editor -> footer.php

---
## virtue with e-commerce
* woocommercd

## Elementor – Parallax Section Background Image
* Code Snippets 

[tutorial for Parallax Effect](https://snifflevalve.com/elementor-tutorials/elementor-parallax-section-background-image/)
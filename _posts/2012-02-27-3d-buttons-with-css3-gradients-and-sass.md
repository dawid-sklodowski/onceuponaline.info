---
layout: post
title: 3D Buttons with CSS3 gradients and Sass
published: true
author: Dawid Sklodowski
author_url: http://github.com/dawid-sklodowski
author_avatar: http://www.gravatar.com/avatar/073c19a8d1fd30baa6dba34eaa55fe90.png
---
Visiting various pages on the web, we often see nice buttons looking like they are 3d.
In most cases those buttons are being made with image – gradients.
CSS3 allows us to create gradients without images, speeding up development and page-loading time.
Another important factor is their flexibility to change color and ability to scale with their containers.
Trying other colors for image-based buttons means changing colors of corresponding gradient-images,
what surely is a time-consuming process. 

![buttons](/images/2013-05-01/buttons.png)

<!--more-->

CSS gradients syntax is a bit cumbersome. So are brightness calculations for different shades of color.
To address this matter, I’ve come up with Sass mixin, which takes base color as argument:
{% highlight sass%}
  @mixin gradient-background($base_color)
    $color1: adjust-hue(lighten($base_color, 7%), 2)
    $color2: darken($base_color, 4%)
    $color3: darken($base_color, 3%)
    $color4: darken($base_color, 12%)
    background-color: $base_color
    background-image: linear-gradient(top,
                                      $base_color,
                                      $color1 45%,
                                      $color2 55%,
                                      $color3 90%,
                                      $color4)
    background-image: -o-linear-gradient(top,
                                         $base_color,
                                         $color1 45%,
                                         $color2 55%,
                                         $color3 90%,
                                         $color4)
    background-image: -ms-linear-gradient(top,
                                          $base_color,
                                          $color1 45%,
                                          $color2 55%,
                                          $color3 90%,
                                          $color4)
    background-image: -webkit-linear-gradient(top,
                                              $base_color,
                                              $color1 45%,
                                              $color2 55%,
                                              $color3 90%,
                                              $color4)
    background-image: -moz-linear-gradient(top,
                                           $base_color,
                                           $color1 45%,
                                           $color2 55%,
                                           $color3 90%,
                                           $color4)
    background-image: -webkit-gradient(linear, 0 0, 0 100%,
                                       from($base_color),
                                       color-stop(0.45, $color1),
                                       color-stop(0.55, $color2),
                                       color-stop(0.9, $color3),
                                       to($color4))
{% endhighlight %}

Mixin for rounded corners is also handy:

{% highlight sass %}
  @mixin border-radius($radius)
    -moz-border-radius: $radius
    -webkit-border-radius: $radius
    -khtml-border-radius: $radius
    -o-border-radius: $radius
    border-radius: $radius
{% endhighlight %}

Having those mixins in place, styling a button is much easier. Style for green button (#77c62f set as base color):

{% highlight sass %}
  a.button
    font-family: verdana
    color: #FFF
    @include border-radius(8px)
    @include gradient-background(#77c62f)
    text-shadow: 1px 1px 2px #000
    font-size: 40px
    padding: 5px 60px
    text-align: center
    text-decoration: none
    font-weight: bold
{% endhighlight %}

Style for red button when hover, active and focus state (#e6572f set as base color):

{% highlight sass %}
  a.button
    &:hover, &:active, &:focus
      @include gradient-background(#e6572f)
{% endhighlight %}

This whole sass code compiles to css file:

{% highlight css %}
a.button {
  font-family: verdana;
  color: white;
  -moz-border-radius: 8px;
  -webkit-border-radius: 8px;
  -khtml-border-radius: 8px;
  -o-border-radius: 8px;
  border-radius: 8px;
  background-color: #77c62f;
  background-image: linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -o-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -ms-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -webkit-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -moz-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -webkit-gradient(linear, 0 0, 0 100%, from(#77c62f), color-stop(0.45, #84d346), color-stop(0.55, #6db62b), color-stop(0.9, #70ba2c), to(#599523));
  text-shadow: 1px 1px 2px black;
  font-size: 40px;
  padding: 5px 60px;
  text-align: center;
  text-decoration: none;
  font-weight: bold; }

a.button:hover, a.button:active, a.button:focus {
  background-color: #e6572f;
  background-image: linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -o-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -ms-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -webkit-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -moz-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -webkit-gradient(linear, 0 0, 0 100%, from(#e6572f), color-stop(0.45, #ea764f), color-stop(0.55, #e4481d), color-stop(0.9, #e44c21), to(#c13c17)); }
{% endhighlight %}

We can use a simple ```<a>``` tag to insert our shiny button into web page:
{% highlight html %}
  <a class="button" href="http://www.google.pl">Button</a>
{% endhighlight %}

Unfortunately CSS3 gradients are available only for FireFox, Chrome, Safari. However they are perfect for mobile devices like iPhone or devices with Android.

<style>
.css-3d-button {
  font-family: verdana;
  color: white !important;
  -moz-border-radius: 8px;
  -webkit-border-radius: 8px;
  -khtml-border-radius: 8px;
  -o-border-radius: 8px;
  border-radius: 8px;
  background-color: #77c62f;
  background-image: linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -o-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -ms-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -webkit-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -moz-linear-gradient(top, #77c62f, #84d346 45%, #6db62b 55%, #70ba2c 90%, #599523);
  background-image: -webkit-gradient(linear, 0 0, 0 100%, from(#77c62f), color-stop(0.45, #84d346), color-stop(0.55, #6db62b), color-stop(0.9, #70ba2c), to(#599523));
  text-shadow: 1px 1px 2px black;
  font-size: 40px;
  padding: 5px 60px;
  text-align: center;
  text-decoration: none;
  font-weight: bold; }
.css-3d-button:hover, .css-3d-button:active, .css-3d-button:focus {
  background-color: #e6572f;
  background-image: linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -o-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -ms-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -webkit-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -moz-linear-gradient(top, #e6572f, #ea764f 45%, #e4481d 55%, #e44c21 90%, #c13c17);
  background-image: -webkit-gradient(linear, 0 0, 0 100%, from(#e6572f), color-stop(0.45, #ea764f), color-stop(0.55, #e4481d), color-stop(0.9, #e44c21), to(#c13c17)); }
.syntaxhighlighter .toolbar{display: none !important;}
</style>

Below is a real example of CSS3 3D button, so you can see how it looks in your browser:

&nbsp; <a class="css-3d-button" href="http://onceuponaline.info">Button</a>

----

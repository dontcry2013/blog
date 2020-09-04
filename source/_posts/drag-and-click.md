---
title: drag and click
date: 2020-09-04 20:31:46
categories:
tags:
---
# 

<!--more-->

``` html
<!DOCTYPE html>
<html lang="zh-CN">
<meta charset="UTF-8" />
<meta view-port="width=device-width, initial-scale" />
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.5.0/css/all.css">

<head>
  <style>
    #box1,
    #box2,
    #box3,
    #box4,
    #box5,
    #box6 {
      position: absolute;
    }

    * {
      margin: 0;
      padding: 0;
    }

    body {
      font-size: 100%;
      margin: 3em;

    }

    h2,
    p {
      font-size: 100%;
      font-weight: normal;
    }

    ul,
    li {
      list-style: none;
    }

    ul {
      overflow: hidden;
      padding: 3em;
    }


    ul li a {
      text-decoration: none;
      color: #000;
      background: #F1F1F1;
      display: block;
      height: 350px;
      width: 280px;
      padding: 1em;
      /* Firefox */
      -moz-box-shadow: 5px 5px 7px rgba(20, 20, 20, 1);
      /* Safari+Chrome */
      -webkit-box-shadow: 5px 5px 7px rgba(20, 20, 20, .5);
      /* Opera */
      box-shadow: 5px 5px 7px rgba(20, 20, 20, .5);
      -webkit-transform: rotate(-6deg);
      -o-transform: rotate(-6deg);
      -moz-transform: rotate(-6deg);
      -moz-transition: -moz-transform .15s linear;
      -o-transition: -o-transform .15s linear;
      -webkit-transition: -webkit-transform .15s linear;
    }

    ul li {
      margin: 1em;
      float: left;
    }

    ul li h2 {
      font-size: 140%;
      font-weight: bold;
      padding-bottom: 20px;
    }

    ul li p {
      font-size: 120%;
    }

    .contact-info {
      position: fixed;
      top: 40%;
      z-index: 99999;
      left: 0;

    }

    .option {
      cursor: pointer;
      position: relative;
    }

    .option i {
      display: block;
      width: 60px;
      text-align: center;
      height: 60px;
      line-height: 60px;
      background: #f1f1f1;
      color: #b9b9b9;
      font-size: 20px;
    }

    .option:hover i {
      color: #6F7784;
    }

    .text {
      position: absolute;
      height: 60px;
      width: 120px;
      background: #6F7784;
      top: 0;
      z-index: -1;
      left: -140px;
      color: #f2e9e1;
      line-height: 60px;
      text-align: center;
      transition: left 0.6s;
    }

    .option:hover .text {
      left: 60px;
    }
  </style>

  <!-- not working javascript -->
  <script type="text/javascript">
    window.onload = function () {
      var box1 = document.getElementById("box1");
      var box2 = document.getElementById("box2");
      var box3 = document.getElementById("box3");
      var box4 = document.getElementById("box4");
      var box5 = document.getElementById("box5");
      var box6 = document.getElementById("box6");
      drag(box1);
      drag(box2);
      drag(box3);
      drag(box4);
      drag(box5);
      drag(box6);

      var noclick = false;
      function drag(obj) {
        console.log(123);
        obj.onmousedown = function (event) {
          obj.setCapture && obj.setCapture();
          event = event || window.event;

          var ol = event.clientX - obj.offsetLeft;
          var ot = event.clientY - obj.offsetTop;

          document.onmousemove = function (event) {
            noclick = true;
            event = event || window.event;
            var left = event.clientX - ol;
            var top = event.clientY - ot;
            obj.style.left = left + "px";
            obj.style.top = top + "px";
            return false;
          };

          document.onmouseup = function (e) {
            console.log(34234)
            document.onmousemove = null;
            document.onmouseup = null;
            obj.releaseCapture && obj.releaseCapture();
            return false;
          };
          
          return false;
        };
        obj.onclick = function (event){
          if(noclick){
            noclick = false;
            event.preventDefault();
            return false;
          }
        }
      }

    }

  </script>

</head>
<!-- Set the main content -->
<!-- Adding a diary container -->
<ul>
  <li>
    <a href="javascript:alert('Hi, you clicked me!');" id="box1">
      <h2>Wednesday 6th September, 10:36pm</h2>
      <p>1, I have continued trade talks with Iran.
        <br>
        2, Replied to Trump about stuff.
        <br>
        3, Followed up on trade deals with Israel.
        <br>
        4, Started hopeful trade deals with Egypt.
        <br>
        <br>
        &nbsp; &nbsp; ADD &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DELETE
      </p>
    </a>
  </li>
  <li>
    <a href="javascript:alert('Hi, you clicked me!');" id="box2">
      <h2>Wednesday 6th September, 7:22pm</h2>
      <p>
        Things to do...
        <br>
        1, follow up trade deals and Arms deals in case people have forgotten i.e., Israel, Turkey, Libya and Egypt
        <br>
        <br>

        <br>
        <br>
        &nbsp; &nbsp; ADD &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DELETE
      </p>
    </a>
  </li>
  <li>
    <a href="javascript:alert('Hi, you clicked me!');" id="box3">
      <h2>Monday 4th September, 5:09pm</h2>
      <p>Important: We decided to have a meeting at 7pm Tuesday to discuss Israel-Palestine conflict.
        <br>
        <br>
        <br>
        <br>
        <br>
        <br>
        &nbsp; &nbsp; ADD &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DELETE
      </p>
    </a>
  </li>
  <li>
    <a href="javascript:alert('Hi, you clicked me!');" id="box4">
      <h2>Sunday 3rd September, 5:37pm</h2>
      <p>
        1, I have assured Morocco that we support them in their ownership of the Western Sahara.
        <br>
        2, Also, have tweeted about it and contacted all media issuing a statement that we support Morocco
        <br>
        <br>
        &nbsp; &nbsp; ADD &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DELETE
      </p>
    </a>
  </li>
  <li>
    <a href="javascript:alert('Hi, you clicked me!');" id="box5">
      <h2>Saturday 2nd September, 4:56pm</h2>
      <p>I have continued trade talks with Saudi Arabia, Israel and Morocco. No response yet. I have also added the
        trade talks tags to each of the emails and archived them.
        <br>
        <br>
        <br>
        &nbsp; &nbsp; ADD &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DELETE
      </p>
    </a>
  </li>
  <li>
    <a href="javascript:alert('Hi, you clicked me!');" id="box6">
      <h2>Saturday 2nd September, 2:48pm</h2>
      <p>I'll do Algeria, Libya and Iran
        <br>
        you can do Saudi, Israel and Morocco
        <br>
        then we need to talk more with the European leaders let's try start talks with them about Syria and say we don't
        believe a ceasefire will work
        <br>
        <br>
        &nbsp; &nbsp; ADD &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DELETE
      </p>
    </a>
  </li>

</ul>

<!-- set a side bar -->
<div class="contact-info">
  <div class="option">
    <i class="fa fa-home"></i>
    <a class="text" href="#">Home</a>
  </div>

  <div class="option">
    <i class="fa fa-envelope"></i>
    <a class="text" href="email">Email</a>

  </div>

  <div class="option">
    <i class="fab fa-twitter-square"></i>
    <a class="text" href="tweet">Tweet</a>

  </div>

  <div class="option">
    <i class="fas fa-newspaper"></i>
    <a class="text" href="news">News</a>
  </div>

  <div class="option">
    <i class="fas fa-comment-alt"></i>
    <a class="text" href="chat">Chat</a>
  </div>

  <div class="option">
    <i class="fa fa-cog"></i>
    <a class="text" href="control">Control</a>

  </div>
</div>

</html>

```
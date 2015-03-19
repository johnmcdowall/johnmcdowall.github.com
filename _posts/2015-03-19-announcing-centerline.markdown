---
layout: post
published: true
title: Announcing Centerline
description: A handy bookmarlet that lets you check the center alignment when designing in the Browser.
---

I like to design in the Browser. Sometimes I'm not quite sure if something is fully centered in the Browser window or not, so I made Centerline. Centerline
is just a dumb bookmarklet that adds a global 'center line' in your Browser viewport. Here it is:

<a href="javascript:(function()%7Bvar%20v%3D%272.1.3%27%3B%20if%20(window.jQuery%3D%3D%3Dundefined%20%7C%7C%20window.jQuery.fn.jquery%20%3C%20v)%7Bvar%20done%3Dfalse%3B%20var%20script%3Ddocument.createElement(%27script%27)%3B%20script.src%3D%27%2F%2Fajax.googleapis.com%2Fajax%2Flibs%2Fjquery%2F%27%20%2B%20v%20%2B%20%27%2Fjquery.min.js%27%3B%20script.onload%3Dscript.onreadystatechange%3Dfunction()%7Bif%20(!done%20%26%26%20(!this.readyState%20%7C%7C%20this.readyState%3D%3D%27loaded%27%20%7C%7C%20this.readyState%3D%3D%27complete%27))%7Bdone%3Dtrue%3B%20initCenterLine()%3B%7D%7D%3B%20document.getElementsByTagName(%27head%27)%5B0%5D.appendChild(script)%3B%7Delse%7BinitCenterLine()%3B%7Dfunction%20initCenterLine()%7B(window.centerLine%3Dfunction()%7B%24(%27body%27).append(%27%3Cdiv%20class%3D%22centerLineDiv%22%3E%3C%2Fdiv%3E%27)%3B%20%24(%27body%27).append(%27%3Cstyle%20type%3D%22text%2Fcss%22%3E.centerLineDiv%7Bcontent%3A%22%22%3B%20position%3A%20fixed%3B%20top%3A%200%3B%20bottom%3A%200%3B%20left%3A%2050%25%3B%20border-left%3A%202px%20dotted%20%23444%3B%20z-index%3A%2099999999%3B%7D%3C%2Fstyle%3E%27)%3B%20%24(%27body%27).append(%27%3Cstyle%20type%3D%22text%2Fcss%22%3E.container%7Bborder-left%3A%201px%20solid%20red%3B%20border-right%3A%201px%20solid%20red%3B%7D%3C%2Fstyle%3E%27)%3B%7D)()%3B%7D%7D)()%3B">CenterLine</a>

Just drag it onto your bookmark bar, and when you need that center line, click the hell out of it. Check it out:

<img src='/images/posts/centerline.png' width="100%" />

To remove, just refresh the page.
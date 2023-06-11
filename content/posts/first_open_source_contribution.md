+++
title = "Making My First Open-Source Contribution"
description = "How I made my first OSS contribution while setting up my website"
date = "2023-06-11"
+++

# Intro

The title is a bit devious – it’s not my first open-source contribution per se. I have made a couple of documentation fixes but those don’t count. I just made my first proper, meaningful open-source contribution on June 9th, 2023. This is a small but still meaningful step for me in the right direction, as someone who wants to get going with making significant open-source contributions in the near future. 

My contribution to the theme I chose for my website was – I fixed a small issue on the light mode of the theme, and integrated a light-and-dark mode toggle button. Before diving into the nitty-gritty, let's go through some recent events that led to my contribution.

I wanted to setup a personal website to use as a space for publishing my thoughts and ideas. Also, I felt that it was about time I setup a site after all. I have recently been learning Rust and have been enjoying the process so I thought I should go with a Rust-based implementation. Upon some basic research I found out about Zola, a Static Site Generator that is fast and easy to get going with, so the choice wasn’t a difficult one.

# Selecting the Theme and Making Changes 

So with my decision of using Zola finalized, I looked at the themes list on the site, and there’s one thing I must concede – the number of themes on display is not a lot. I found a couple of themes that I liked and decided to explore the one I found the most appealing as well as content-focused - [Apollo](https://github.com/not-matthias/apollo "https://github.com/not-matthias/apollo") which is what the site’s current theme is based on.

On fiddling around with the its code, I found out that the social icons weren't loading properly for the light mode. It used an inversion filter to invert their colours from black to white for the site’s dark mode, but the property was active even with the light mode.

So I opened an issue regarding the same and forked the repository in the meantime. I also found [Archie-Zola](https://github.com/XXXMrG/archie-zola/ "https://github.com/XXXMrG/archie-zola/"), the theme on which Apollo is based, to have a pleasing neon-green primary aesthetic that I liked more. I also found out that it had a dark and light mode toggle button, which I then wanted to implement in my own fork as well. So I began working on making these changes. 

I fixed the problem with the icons’ disappearance relatively quickly, I just needed to move the relevant logic from the main file to the dark mode’s SASS file. Properly integrating the theme toggle button was much more complex and it required me to spend some time understanding the main visual logic of the two themes, and then writing the code, helping me learn about some core frontend styling logic. 

# Making The Contribution 

My talks with Apollo's creator by this time had also gone well. He encouraged me to open a Pull Request. I then briefly updated him about my fork and asked him whether he would like to have the toggle button in Apollo and he had a positive response again. 

I then created a new branch in my fork, made the requisite changes and opened the [PR](https://github.com/not-matthias/apollo/pull/20 "https://github.com/not-matthias/apollo/pull/20").  It was reviewed and merged promptly, meaning I had made my first proper open-source contribution. 

# My Experience 

Although this is just a small instance, I had a lot of fun with the whole thing. I had to read up on several less-explored topics. I also got familiar with how the theme operates internally, which has led to me making more tweaks for my website. 

I hope this can serve as encouragement for anyone reading this post, to take a look at open-source software and possibly support or take part in the development process. 

My plan now is to delve deeper into Rust, learn more about the language and hopefully make more contributions :D
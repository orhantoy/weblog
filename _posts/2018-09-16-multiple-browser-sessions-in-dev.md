---
title: Browser sessions in development
date: 2018-09-16 15:00:00 +0200
categories: rails puma browser dev
---

It is a bit of a pain to test something out in development where you want isolated sessions. It is a common need to log in with two different users - the alternative being that you log in with the first user, do something, log out, log in with the other user, and then do something. Many people solve this by having a regular browser session and another incognito mode session. With [Puma-dev](https://github.com/puma/puma-dev) this can be done pretty easily.

I've come to enjoy using Puma-dev for Rails apps I work on regularly.
Basically you would access an app called `myapp` by going to `http://myapp.test`.
<br>
An added benefit over using `rails s` and accessing your app on `http://localhost:3000` is that Puma-dev handles subdomains. So in addition to `http://myapp.test` you can also access your app via `http://www.myapp.test`, `http://www2.myapp.test`, etc.
And because each subdomain by default will have separate cookies we get the ability to have multiple sessions open in the same browser. Recalling the common need of multiple users, you could log the first user in via `http://www1.myapp.test` and the other user via `http://www2.myapp.test` - all in the same browser window.

Easy! 🎉

---
layout: post
title:  "지킬 사용법"
date:   2020-09-03 14:31:29 +0900
categories: Diary
comments: true
---

※ 지킬 사용법이 담긴 포스팅이라 원문에 덧붙여서 보존해놓겠다.
<br/>

## 잊어버리기 쉬운 명령어 및 사용법

1. 수정사항을 localhost상에서 실시간으로 확인하는 명령어
```
bundle exec jekyll serve
```
2. git에 commit하고 push하는 명령어
```
git add .
git commit -m "blabla"
git push
```
<br/>

## 원문
<br/>

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

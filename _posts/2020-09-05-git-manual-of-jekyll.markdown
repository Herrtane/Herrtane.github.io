---
layout: post
title:  "Jekyll 사용법"
date:   2020-09-05 01:00:29 +0900
categories: Git/Github
comments: true
---

# 잊어버리기 쉬운 명령어 및 사용법

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

3. (원문) Jekyll also offers powerful support for code snippets:

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

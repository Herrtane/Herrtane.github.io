---
layout: post
title:  "Jekyll 사용법"
date:   2020-09-05 12:00:29 +0900
categories: Git/Github
comments: true
---

# 잊어버리기 쉬운 명령어 및 사용법

1. 수정사항을 localhost상에서 실시간으로 확인하는 명령어
```
bundle exec jekyll serve
```
2. git을 시작하고, git에 commit하고 push하는 명령어
```
git init
git add .
git commit -m "blabla"
git push origin master
(master는 현재 사용하는 기기의 branch를 뜻한다.)
```
3. github과 연동하는 명령어
```
git remote add origin https://github.com/[이름]/repository
(계속 언급되고 있는 origin은 repository이름을 말하며, 꼭 origin이라고 작성할 필요 없이 편한 내용으로 설정하면 된다. 앞으로 origin부분에 해당하는 이름을 사용해서 해당 repository에 접속할 수 있게 된다.)
git remote remove origin
```

4. github에서 local로 내려받는 명령어
```
git pull origin master
(보통 pull을 할 때는 로그인을 해야될 때가 많은데, 귀찮더라도 직접 입력하도록 하자. 파일로 저장해서 로그인 과정을 생략할 수도 있지만, 보안에 취약해진다. 특히, 나처럼 main computer외에 moblie이나 virtual machine을 사용할 경우 일일이 로그인을 하는 것이 좋겠다.)
git clone [저장소 주소]
(이 경우 완전히 처음 서버의 프로젝트를 다운받을 때 주로 사용한다. 자동으로 init까지 해준다.)
```

5. (원문) Jekyll also offers powerful support for code snippets:

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

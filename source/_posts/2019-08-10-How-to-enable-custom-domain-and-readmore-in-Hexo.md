---
title: How to enable custom-domain and readmore in Hexo
date: 2019-08-10 09:21:06
tags:
---

What's is `Hexo` ?

according to the official introduction its a blog framework, have more than 20k forks and large user base.

> “A fast, simple & powerful blog framework”

The deployment/usage is quite strightforward but not that easy to find ways to manage the custom domain in github pages and the "Read More" in [minos](https://github.com/ppoffice/hexo-theme-minos) theme

The website you are visting is built on top Hexo and Minos theme and hosting in github pages with a CNAME for dns resolvtion

### How we made it
<!---more--->

1. Generate the CNAME file in source directory
```SHELL
echo "ACTUAL Website DOMAIN" > source/CNAME
```

**if the site is deployed to github before without the CNAME file, need to delete `public` folder otherwise the change won't take into effect**
then update your dns server make it point to the desied github site, for me its

```SHELL
$ dig www.ystacks.com

; <<>> DiG 9.11.3-1ubuntu1.8-Ubuntu <<>> www.ystacks.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42551
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ystacks.com.		IN	A

;; ANSWER SECTION:
www.ystacks.com.	1498	IN	CNAME	jiangytcn.github.io.
jiangytcn.github.io.	1498	IN	A	185.199.109.153
jiangytcn.github.io.	1498	IN	A	185.199.110.153
jiangytcn.github.io.	1498	IN	A	185.199.108.153
jiangytcn.github.io.	1498	IN	A	185.199.111.153
```

2. Enable the ReadMore for the posts

add `<!---more --->` section in the post articles, and whatever above it would be present as the abstract in the list page like below

![articles with readmore](/images/article-list-readmore.png)

That's it.


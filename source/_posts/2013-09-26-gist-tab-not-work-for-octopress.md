---
layout: post
title: "在octopress中gist tab不能正确的插入gist代码"
date: 2013-09-26 00:09
comments: true
tags: [Github, Ruby]
---

今天尝试用Octopress的[gits tab](http://octopress.org/docs/plugins/gist-tag/)插件来把gist插入到博客中，但是发现没有插入成功，调用```rake generate```报如下的错误：

`Gist replied with 404 for https://raw.github.com/gist/6700691/ClassHierarchy.cpp`

我看了一下这个url，确实是404。于是就到了自己的gist页面看了看，发现应该用如下的url：  
`https://gist.github.com/6700691#ClassHierarchy.cpp`

但是其实这个url也会重定向到：

`https://gist.github.com/fresky/6700691#ClassHierarchy.cpp`

于是就把`plugins\gist_tag.rb`的代码改了改，改成如下：

```ruby
    def get_gist_url_for(gist, file)
      "https://gist.github.com/#{gist}##{file}"
    end
```

另外如果html返回302，不报异常：

```ruby
    def get_gist_from_web(gist, file)
      gist_url          = get_gist_url_for gist, file
      raw_uri           = URI.parse gist_url
      proxy             = ENV['http_proxy']
      if proxy
        proxy_uri       = URI.parse(proxy)
        https           = Net::HTTP::Proxy(proxy_uri.host, proxy_uri.port).new raw_uri.host, raw_uri.port
      else
        https           = Net::HTTP.new raw_uri.host, raw_uri.port
      end
      https.use_ssl     = true
      https.verify_mode = OpenSSL::SSL::VERIFY_NONE
      request           = Net::HTTP::Get.new raw_uri.request_uri
      data              = https.request request
      if data.code.to_i != 200 and data.code.to_i != 302
        raise RuntimeError, "Gist replied with #{data.code} for #{gist_url}"
      end
      data              = data.body
      cache gist, file, data unless @cache_disabled
      data
    end
  end
```

这样gist就能正确的插入了，但是显示还有些问题，代码字体太大，和行数不能匹配，这个下次有机会在看看吧。显示效果可以参看[C++程序在debug模式下遇到Run-Time Check Failure #0 - the Value of ESP Was Not Properly Saved Across a Function Call问题](/2013/09/11/why-have-Run-Time-Check-Failure-0-The-value-of-ESP-was-not-properly-saved-across-a-function-call-error/)。

完整代码参加[gist](https://gist.github.com/fresky/6702098)。

---
title: Fail to build gem native extension.
---

뒤늦게 mac os를 High Siera 에서 Mojave로 업데이트 한 후, 갑자기 jekyll 로컬 실행이 되지 않았다.
일단 bundler로 gem 의존성을 다시 업데이트 하라는 말에, 아래 명령어를 실행했다.

> **Bundler?**  
> 번들러는 필요한 정확한 gem과 버전을 추적하고 설치하여 루비 프로젝트를 위한 일관된 환경을 제공합니다.
> 번들러는 의존성 지옥에서 벗어나게 하고, 필요한 gem이 개발, 스테이징, 프로덕션에 있는지 확인해 줍니다.
>
> 프로젝트 최상위 디렉터리에 있는 Gemfile에는 해당 프로젝트의 의존성을 나타냅니다.
>
> (출처 : http://ruby-korea.github.io/bundler-site/)


```bash
$ bundle install
```

그런데 에러가 난다.

```
...(생략)

Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

...

An error occurred while installing eventmachine (1.2.7), and Bundler cannot continue.
Make sure that `gem install eventmachine -v '1.2.7' --source 'https://rubygems.org/'` succeeds before
bundling.

...(생략)

```

좀 더 자세히 보니 `mkmf.rb` 파일의 헤더를 찾지 못한다는 내용을 확인할 수 있었다.

```

mkmf.rb can't find header files for ruby at
/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/include/ruby.h

```


해결 방법은 xcode command line tool을 설치하는 것으로 끝났다. 업데이트 후 xcode 관련 설정이 초기화 되었던 것 같기도 하고...?

```bash
$ sudo xcode-select --install
```

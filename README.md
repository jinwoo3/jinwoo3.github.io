# Authors(User) 추가하는 법

content/authors 디렉토리 내에 markdown 파일 제목을 [닉네임.md]에 맞추어 만든 후

```
---
id: lloyd
title: 'Lloyd Shin'
link: 'your-github-link'
image: 'uploads/author-images/author-avatar.png' (따로 넣고싶은 경우엔 static/uploads/author-images 내에 사진을 넣어주신 뒤 해당 파일 이름 기입)
email: 'your-email'
blurb: 'description'
---

# Bio

hi
```

위 양식에 맞추어 내용 작성해주시면 됩니다.

# Post 작성하는 법

content/posts 디렉토리 내에 markdown 파일 제목을 [yyyy-mm-dd-name.md]에 맞추어 만든 후

```
---
title: 제목 적기
slug: url-eng-number (only eng, number)
author: lloyd (authors 디렉토리 내 자신의 마크다운 파일에서 id를 적으시면 됩니다)
tags:
  - tag example1
  - tag example2
excerpt: description dd
date: yyyy-mm-dd
featuredImage: thumbnail image url
---
```

위 내용을 적어주신 뒤 내용 작성하시면 됩니다.

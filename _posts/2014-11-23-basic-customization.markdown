---
layout: post
title: "Octopress 3.0 / Jekyll Basic Customization"
keywords:
description:
thumbnail:
facebook_type:
facebook_image:
---

A few simple configs to change after an initial install

`_config.yml`

**description**

`description: > # this means to ignore newlines until "baseurl:"`

 `Change your description here.
 You can make it multi-line as described in the comment above.`

**permalinks**
`permalink: /blog/:year/:title`

**pagination**

  ```bash
  paginate: 5
  paginate_path: "blog/page:num/"
  ```

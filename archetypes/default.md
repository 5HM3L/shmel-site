---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: false
author: ["z0yac"]
categories: [""]
tags: [""]
archives: ["{{ dateFormat "2006" .Date }}", "{{ dateFormat "2006-01" .Date }}"]
eyecatch: "/images/og/{{ .Name }}.png"
ogimage: "/images/og/{{ .Name }}.png"
comments: false
adsense: false
description: ""
url: "/{{ .Type }}/{{ .Name }}/"
---
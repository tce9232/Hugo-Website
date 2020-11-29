---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Analyzing Music and Emotion with NLP"
summary: "An examination of the changing landscape of song lyric emotion, and how different artists and genres compare"
authors: ['Ted Carlson']
tags: ['Natural Language Processing', 'Data Visualization']
categories: []
date: 2020-11-28T13:41:02-06:00

# Optional external URL for project (replaces project detail page).
external_link: ""

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Death Grips playing a presumably noisy and angry show"
  focal_point: ""
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

{{< load-plotly >}}

### Pop, Metal and Hip Hop Emotions


{{< plotly json="/plotly/3DLyricScatter.json" height="600px" width="800px"  >}}

### Rock Music Emotions

{{< plotly json="/plotly/3DRockLyricScatter.json" height="600px" width="800px"  >}}

### Rap Music Emotions

{{< plotly json="/plotly/3DRapLyricScatter.json" height="600px" width="800px"  >}}

### Test

{{< plotly json="/plotly/Value-Properties-Scatter.json" height="600px" width="800px"  >}}


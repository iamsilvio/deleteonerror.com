---
title: "Blackout PDF on MacOS using Preview"
subtitle: ""
date: 2020-02-14T01:49:32+01:00
draft: true
---

If you want to share documents and want to prevent that others can read the text you want to hide you have to invest some extra work.

1. open the pdf file in preview and use the annotate tools to put black rectangles on top of the text you want to hide
2. Export the file as tiff `File -> export ... -> Tiff` (this will be a large file) but it ensures that the OCR data from the pdf is striped
3. open the tiff in Preview and export it as PDF

All OCR content and the text behind the Blackout blocks will not be persisted as data in the pdf anymore

have fun and enjoy your privacy and freedom

+++
# A "place": restaurant, bar, shop, attraction, trail... Fill what you know, delete the rest.
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
date = '{{ .Date }}'
draft = true                     # set to false to publish
summary = 'One sentence shown in lists.'
weight = 50                      # lower = higher in the list (10, 20, 30...)
tags = []                        # e.g. ['pizza', 'family', 'late-night']

[params]
  address = ''
  distance = ''                  # e.g. '5 min walk' or '2 km'
  phone = ''
  website = ''
  hours = ''                     # e.g. 'Mon–Sun 11:00–22:00'
  price = ''                     # '$', '$$', '$$$'
  map = ''                       # Google Maps share link
  sponsored = false              # true for paid listings; shows a small "Sponsored" label
+++

A short, friendly description of the place and why a guest might like it.

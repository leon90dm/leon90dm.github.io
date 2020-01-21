---
layout:     post
title:      "Changing font in matplotlib in Linux"
subtitle:   "matplotlib 修改字体"
date:       2020-01-21
author:     "BilboDai"
catalog: true
tags:
    - data analysis
---

1. download desired font

2. move font file to the directory `.local/share/fonts/`. (First create this directory if it doesn't exist). Other fonts may have `.ttf` files instead, but the idea is the same.

   ```bash
   mv STKaiti.ttf .local/share/fonts/
   ```

3. Clear and rebuild the font cacehe.

   ```bash
   fc-cache -f -v
   ```

   >install font config using  `# yum -y install fontconfig` if  `fc-cache` command is not found.

4. Get matplotlib to recognize the new font. To do this start, a python interpreter (such as ipython), and run:

   ```python
   from matplotlib.font_manager import _rebuild; _rebuild()
   ```

5. To see the list of fonts that are available (and to make sure that it includes the Libertine fonts), run:

   ```python
   for font in  font_manager.findSystemFonts():
       print(fonts)
   ```

   Output should looks like:

   ```python
   ['/usr/share/fonts/dejavu/DejaVuSans-Oblique.ttf',
    '/home/latila/.local/share/fonts/STKaiti.ttf']
   ```

6. Change font using code snippet below:

   ```python
   def change_font(font_name):
       import matplotlib as mpl
       import matplotlib.font_manager as font_manager
       #font_manager._rebuild() #rebuild in case of not effective
       fontpath = [x for x in matplotlib.font_manager.findSystemFonts(fontpaths=None, fontext='ttf') if font_name in x]
       prop = font_manager.FontProperties(fname=fontpath[0])
       mpl.rcParams['font.family'] = prop.get_name()
       mpl.rcParams['axes.unicode_minus']=False
   ```

   Run `change_font('STKaiti')` then you are good to go!












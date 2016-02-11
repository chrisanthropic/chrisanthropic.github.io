---
layout: post
title: "Bundling Your Digital Comics"
date: 2014-12-02T17:10:14-08:00
---

I highly recommend using [ImageMagick](http://www.imagemagick.org/) and [mogrify](http://www.imagemagick.org/script/mogrify.php) for most of these tasks. Make sure you understand how to use these tools before running the commands since they will modify your images. All of the commands should be run from the terminal and you should be in the directory of files that you want to modify.

### Preparing your Images
1. Make sure your images are using the RGB colorspace since we're not printing them out
2. The images can be any size but if they're not all the same size then opening/reading the comic will be slow and that sucks. I like 750x1150
  * You can resize all of your images using the `mogrify` command-line tool. If all of your images are .jpg, open a command line, cd to the location of your images and use the command below  
  `mogrify -resize 750 *.jpg`
  * This command says to use `mogrify` to resize all .jpg files to 750 width while keeping the original aspect ratio
3. You can use .png images but .jpg tend to be smaller
  * You can quickly convert all of your .png files to .jpg with the following command:
  `mogrify -format png *.jpg`
4. Reduce the pixel density of your images to 72 ppi since that's what most computer/phone/tablets use (all but Pixel displays)
  `mogrify -density 72 -set units PixelsPerInch *.jpg`
5. Compress your images
  `mogrify -strip -interlace Plane -quality 85% *.jpg`

### Create a CBZ
A .cbz file is nothing more than a .zip file of your comic pages that is renamed from .zip to .cbz

1. Create a new folder with the name of your comic
2. Copy all of your comic pages into the folder
3. Create a .zip of the folder
4. Change the .zip extension to .cbz
5. Congrats, you made a CBZ

### Create a CBR
A .cbr file is nothing more than a .rar file of your comic pages that is renamed from .rar to .cbr

1. Follow the steps for CBZ but instead of making a .zip of the folder, make a .rar
2. Rename the .rar to .cbr
3. Boom, done.

### Create a PDF
1. Convert your folder of images to a multi-page pdf
`convert *.jpg Comic.pdf`

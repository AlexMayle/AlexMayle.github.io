---
layout: post
title: Finding All the Objects in an image, Without Looking at the Whole Image
tags: [bounding, box, boxes, BBFinder]
categories: [algorithms]
description: Can we place bounding boxes around all the objects in the image, without even looking at the whole image? Yes, yes we can. 
---

p. Last semester I received an interesting assignment. given an image, render a new image with [bounding boxes](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=4&cad=rja&uact=8&ved=0ahUKEwj5qNbx4q7RAhVH6YMKHTElCuAQFggqMAM&url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FMinimum_bounding_box&usg=AFQjCNG9qOZ4aoFsuQD53NgEQCUGOyQkPQ&sig2=n1JHXcFWgQNZFfiNt95W0w) around all of the objects, like so:

!{display:block; margin-left:auto; margin-right:auto;}{{ site.BASE_PATH }}/assets/media/bounding-box-example.png!

### File Format

If we are working with colored images and compressed file types, such as JPEG, this could get complicated, but we have a number of simplifying factors. We are going to be working with black and white images, specifically the [ASCII PBM](https://people.sc.fsu.edu/~jburkardt/data/pbma/pbma.html) format. This is essentially a file containing a 2D array of '1's and '0's, denoting black and white pixels, respectively. Below is an example ASCII PBM file  containing a 'J'.

```P1
# This is an example bitmap of the letter "J"
6 10
0 0 0 0 1 0
0 0 0 0 1 0
0 0 0 0 1 0
0 0 0 0 1 0
0 0 0 0 1 0
0 0 0 0 1 0
1 0 0 0 1 0
0 1 1 1 0 0
0 0 0 0 0 0
0 0 0 0 0 0
```

### Defining "objects" within an image

'We will define an "object" within an image to be a collection of "neighboring" black pixels. Two black pixels are neighboring if the distance between their x coordinates and y coordinates are both less than or equal to 3. Obviously, the 'j' in the image above would be a single object because each black pixel is neighbors with at least one other black pixel, forming a collection.

The brute force approach would be to iterate over each pixel and do some bookkeeping along the way to keep track of neighboring pixels. We can then use the two pixels in each group of neighbors with the minimum and maximum `(x,y)` coordinate pair to denote the top-left and bottom-right corner of the bounding box, respectively. This would be "quick enough," but I ask the question: can we do better? Can we find all the bounding boxes in an image without checking every single pixel? Yes, yes we can.

<div class="alert alert-info" role="alert">
  <p>
    **Aside: Axis-Aligned Bounding Boxes**
  </p>

<p>
  We are actually dealing with AABB's (Axis Aligned Bounding Box) which often arise when transforming objects in graphics programs, or [determining object collisions in computer games](https://en.wikipedia.org/wiki/Collision_detection#Bounding_boxes).
</p>
</div>


### Creating a better algorithm

First, let's examine the nature of these bounding boxes. The box has to enclose, or at least overlap every black pixel in the object. Imagine we have found one black pixel and we are now looking for a neighbor. If we find one to the right, we will have to update the right side of the bounding box to enclose that pixel. In other words, we can only increase the size of the box, never shrink it.

We have just stumbled upon something we can use. Recall that two black pixels are neighbors if the distance between their `x` and `y` coordinate are both less than or equal to 3. If we have a black pixel, and we are looking for a neighbor, we can start by looking 3 pixels away in some direction. If a black pixel exists there, we can update the bounding box. We don't need to look 2 pixels or 1 pixel away because the box can't shrink, the contents of those pixels are irrelevant. If a black pixel doesn't exist 3 pixels away, then we can check 2 pixels away, and so on.

We can now make a simple subroutine to grow the box in this "greedy" fashion. The following algorithm takes as input a bounding box and a pixel that is on one of three sides of the box. The pixel will never be on the top side of the box because of the way we are going to find the first black pixel to grow the box from, which we will talk about later. This subroutine returns the neighboring pixel as soon as one is found, or null if not.

```
// Algorithm to grow a single bounding box
Subroutine: growBox
Input:
    boundingBox: current growing bounding box
    pixel: a pixel on the perimeter of boundingBox
Begin:
    if:   pixel is on right side of boundingBox
        greedy search diagonally southeast
        greedy search to the east
    end if
    else if:   pixel is on bottom side of boundingBox
        greedy search diagonally southeast
        greedy search diagonally southwest
        greedy search south
    end if
    else if:   pixel is on left side of boundingBox
        greedy search diagonally southwest
        greedy search to the west
    end if
End
```

We can see that it always checks diagonally first, because that increases the size of the box the most while visiting the smallest amount of pixels. It also never checks pixels that are already enclosed in the bounding box, as those don't matter anymore.

#### A convenient case

If we used this subroutine on an all black 10x10 image starting from the top-left pixel, we would only visit 4 pixels. The image has 100 pixels, but we only checked 4 before the bounding box took up the whole image and had no more room to grow. That's a **.04** speedup!

#### The general case

Unfortunately, this is only a particularly convenient scenario. The general case will require additional steps in the algorithm.

As mentioned earlier, we need a way to find our first black pixel to try and grow the box from. This will be handled by `getBoxSeed()`. I haven't come up with any way to do this other than searching each pixel in the first row before moving on to the second row. This is inefficient in the sense that for an empty image, we would end up checking each pixel. On the flipside, if we do find a black pixel, we know that the bounding box can never grow vertically upwards, since we already checked every pixel above it. This is why we don't search that direction in the subroutine above.

So what happens when our "greedy" approach to finding neighbors fails? While our algorithm is useful and quick, we are going to need an exhaustive search before the box is said to be done growing with full confidence. This boils down to simply checking the perimeter for neighbors. This is implemented in the `checkPerimeter()` function. It is still greedy in the sense that it will look 3 pixels away before checking 2 pixels away, but we will need to check every pixel within 3 rows or columns of the perimeter before we know the box is done.

So we now have the conceptual algorithm for finding all the bounding boxes in an image.

```
// Algorithm to find all Bounding Boxes in an image
Subroutine: findAllBoxesInPartition
Input:
    image:  represents an ascii pbm file
    boundingBoxes:  list of bounding boxes already found
begin:
    while a black pixel exists outside of all bounding boxes
        pixel := black pixel outside of all bounding boxes
        loop:
            while pixel has neighbor found through growBox()
                pixel := neighbor
            end while
            if checkPerimeter() yields a new neighbor
                pixel := neighbor
            else
                break   // bok is done growing
        end loop:
    end while
end
```

### Results & Complexity

As we saw in specific cases, our algorithm can lead to incredible speedups, but on more realistic images, we are going to see more realistic results. For example, [this image of an odd dragon thing](https://people.sc.fsu.edu/~jburkardt/data/pbma/gerrymander.png) had a speedup of 0.51 and produced the image below.

!{display:block; margin-left:auto; margin-right:auto;}{{ site.BASE_PATH }}/assets/media/bounding-box-result.png!

Computing an uppor or lower bound will be tricky because of the vast degrees of freedom in images: their dimensions, number of objects depicted, the objects' sizes, positioning... However, just as I illustrated an example in which the algorithm excels, I'd like to provide one in which it does not.

!{display:block; margin-left:auto; margin-right:auto;}{{ site.BASE_PATH }}/assets/media/plus.png!

Consider the image above. Growing the bounding boxes to their full sizes would be efficient, but checking their perimeter to make sure they are done growing won't be. The center of the plus resides in the perimeter of all four objects, so it will get checked four times. Each "arm" is on the perimeter of two objects, respectively, so each will get checked twice. In the end it results in a 1.51 speedup -- or what is actually a slow down. 

Of course, we could sacrifice memory in the name of time and keep track of pixels we have checked; however, this does not help us very much. In general, it may, but in our case, it does not. We are keeping our image in a two dimensional array with constant time access. We could use a similar data structure to keep track of whether each pixel has been checked or not. We can then see that it takes the same amount of time to check if a pixel is black or white, as it does to see if the pixel has already been checked for its color.

So for a pixel that hasn’t been checked, we are actually doubling the time it takes to query the color of a pixel. Likewise, if the pixel has been checked, we mite as well have spent the time querying the color of the pixel, because there is no speedup either way. 

If you'd like to mess around with the code yourself and try to speed it up, go clone the repo on GitHub!

[Bounding Box Finder on GitHub](https://github.com/AlexMayle/BBFinder)

Also, I'd be very interested in a lower or upper bound, even if it's loose, so don't hesitate to leave a comment or contact me directly.










---
layout: about
title: about
permalink: /
subtitle: <a href='https://www.cs.cmu.edu/afs/cs/academic/class/15418-s24/www/'>15418</a> Parallel Computer Architecture and Programming Final Project.

profile:
  align: right
  image:
  image_circular: false # crops the image to make it circular
  more_info:

news: false # includes a list of news items
selected_papers: false # includes a list of papers marked as "selected={true}"
social: false # includes social icons at the bottom of the page
---

This project focuses on parallel seam carving, a technique used for content-aware image resizing. We plan to use parallel computing techniques to optimize seam carving processes, aiming for faster and more efficient image resizing. We will be working with parallel computing systems like CUDA and GPUs to distribute computation tasks across multiple processing units, enhancing the speed and scalability of seam carving operations.

View our project proposal [here](https://docs.google.com/document/d/1LakwSzRYJRiu1WjWrBUieWHFqoFiMERCQS3AeOFNL0Y/edit?usp=sharing).

By Jack Aguilar (jackagui) and Michelle Feng (msfeng)


### Milestone Progress Report

| Dates | Work | 
| -------- | -------- | 
| 4/17-4/21     | Parallelize across seams–static (Michelle), dynamic (Jack)     | 
|4/22-4/25| Parallelize across subproblems (together–VSCode Liveshare)|
|4/26-4/28| Focus on performance improvements: Update gradient matrix (don’t recompute every time we remove seams)–Jack, Horizontal seams–Michelle|
|4/29-5/2|Buffer week for improving performance if we fall behind, or working on stretch goals detailed in the “hope to achieve” section|
|5/3-5/5|Complete final project report|
| 5/6 | Presentation|


*Completed Work*
* Image handling: We have spent time finding resources to help handle input images easily. As we were not familiar with these technologies (especially with C++), this took some time to research. Through StackOverflow, we became aware of multiple different open source tools that people often use to handle reading and writing to images in C++. Since this took some time to figure out, we will detail some of our research attempts here:
    * OpenCV is an open source computer vision library that has a C++ implementation. We found many recommendations towards this platform, but this ultimately would have taken some installations that we did not have the time to go through (especially since we wanted to use the Gates clusters for our parallel implementation). Furthermore, OpenCV would have been slightly overkill, since we only want to read and write to images, and this tool would have many complicated, unnecessary features.
    * We landed on using the stb library, which is open source and helps read in images. This ultimately combatted the issues we recognized with OpenCV, since we were able to copy only the necessary headers for our own purposes. The existence of headers also made this relatively easy to run on the Gates clusters.
* Sequential Seam Carving: We separated the skeleton of our sequential code into the following steps:
    * Image handling, including using the stb library that we describe above and converting read images into a 2D matrix of a self-defined Pixel struct (holding red, green, and blue integer values). This matrix is implemented as a C++ standard vector of vectors, which we landed on after realizing that our initial implementation of pixel matrices using 2D arrays did not allow for fast dynamic manipulation (aka efficient removal of seams). This change was important to us because we wanted to minimize unnecessary overhead (like the overhead that would be introduced with removing relatively unpredictable indices within statically allocated arrays).
    * Finding the gradient array. This first involved determining an “energy function” to use to determine the seams. We landed on the gradient due to its familiarity from past classes (but maybe other energy functions would be cool to explore in the future). We implemented functions to calculate differences between surrounding pixels, and the actual gradients of each pixel. This takes place in O(n) time, and has frequent memory accesses per gradient calculation (so per pixel), which is pretty slow and could be explored further.
    * Determining a seam. This involved finding a minimum gradient sum of a path from any pixel in the top row to the current pixel. This must be done row-by-row because a row depends on the row above it, so this sequentially is O(n). Thus, the bottom row of this matrix contains the minimum gradient sums of any path from the top row to the bottom, and so we find the minimum of the values in the bottom row, which corresponds to the minimum cost seam. We then start from the bottom row and propagate upwards, because once we determine the end of the seam, we can start keeping track of the path by working backwards from the end of the path and going to the pixel above with the lowest value in the min cost array.
    * Removing multiple seams. Because once a seam is removed we are removing pixels from the image, certain gradients and min-cost values must be recomputed. Gradients only need to be recomputed along the seam, but changing a gradient value can change the min-cost value of the three adjacent pixels below, each of which can change the value of the three adjacent pixels below each of them, which spreads out to a large triangle shape across the image. Initially we are simply recomputing the entire min-cost matrix, but if we have time will use a smarter algorithm to only recompute the values we need to



At this time, **we do not have preliminary results**. The process that we describe is implemented, but buggy, which we were not able to finish working through because of our involvement with booth these past couple of weeks.

Overall, we are relatively on track with our goals, which did not specify a completed parallel implementation at this point. Since we haven’t been able to evaluate a parallel implementation yet, it’s not as clear to determine how easily we will be able to complete our “nice to have” features. However, we should have a more concrete goal by this time next week! 

*Next steps*
In our next goals, we want to implement solutions that evaluate multiple ways to parallelize this task. We have identified the following methods that we hope to explore for the final demo:

* Parallelizing across seams
    * Statically assigning boundaries to seams by assigning entire columns of the gradient matrix to have value infinity (or the maximum float value). Thus no element along those columns would be chosen in the seam finding function, and we can find seams parallelly on each side of a “boundary.” This would involve a tradeoff with the quality of the resultant photo, which we will analyze once we have the results. 
    * Find a single seam, and then assign each pixel in the seam to have gradient infinity, so that two seams can be found in parallel on either end of the seam, after which this process can be repeated recursively. This should work better than restricting a seam to a vertical stripe of the image, at the cost of lower parallelization.

* Parallelizing across subproblems
    * The sequential algorithm that we implemented relies on a dynamic programming approach, which inherently takes a tree-like structure in terms of relationships between subproblems. Due to this structure, we should be able to parallelize across paths of the tree.

* Comparison to horizontal seams* (nice to have)
    * Our current approach finds vertical seams to remove. It could be interesting to implement a version that finds horizontal seams (without trivially transposing the matrix and running the same algorithm), since performance differences could be reasoned about with differences in memory access patterns. This is something that could be nice to have, but is not necessary.


Because we have experience programming in CUDA and do not require any more features other than the ones we have used in the past, there are not any remaining unknowns. The biggest thing we need to figure out is an efficient way of finding multiple seams at once. Because multiple seams cannot go through the same pixels, you cannot remove multiple seams at once using a naive algorithm, and so finding a method for doing this that doesn’t sacrifice performance but still parallelizes well is the main challenge.

*Poster session plan*
At the poster session, we plan to have a demo (or if it’s too slow, a video of a sped up demo) that shows our sequential and parallel algorithms being run on the same image for the same amount of iterations. After this, we should be able to see the resultant image from both algorithms, as well as the time it took to run each. We will also have a graph ready comparing the number of processors to the time it took for our parallel approach to run. This would be accompanied by an explanation of the performance (discussion of where overhead could be being introduced, memory access analysis, etc.). We will also show the results on some other images, if people are interested.







<!-- Write your biography here. Tell the world about yourself. Link to your favorite [subreddit](http://reddit.com). You can put a picture in, too. The code is already in, just name your picture `prof_pic.jpg` and put it in the `img/` folder.

Put your address / P.O. box / other info right below your picture. You can also disable any of these elements by editing `profile` property of the YAML header of your `_pages/about.md`. Edit `_bibliography/papers.bib` and Jekyll will render your [publications page](/al-folio/publications/) automatically.

Link to your social media connections, too. This theme is set up to use [Font Awesome icons](https://fontawesome.com/) and [Academicons](https://jpswalsh.github.io/academicons/), like the ones below. Add your Facebook, Twitter, LinkedIn, Google Scholar, or just disable all of them. -->

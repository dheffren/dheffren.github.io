---
layout: post
title:  "Camera Geometry"
date:   2025-01-11 13:27:32 -0500
categories: jekyll update
---

Recently, as I've been working on robotics projects, including making my own SLAM system to generate point cloud data of my room (and to locate the camera in the room), I've had to brush up on my Camera Geometry knowledge from my Computer Vision courses in undergrad. 

I remembered this being quite difficult to wrap my head around at the time, and ESPECIALLY difficult to implement oneself: There's a lot of places where things can screw up, and there's more dissonance between math and code than typically expected in Computer Science/ML. 

Thus, I thought it might be a good idea to write up a post introducing people to camera geometry in a hopefully intuitive and enjoyable manner. 

As someone who does Applied Math and Computer Science, I typically understand things best through both math AND code, so I'll be doing that here as well. 


# How does a camera work, anyway? 
It's easy to take the development of camera technology for granted. In 200 years, we went from having to commission an official portrait of your family, to being able to take a picture on our phone in a couple seconds. The capacity to capture a transfixed moment in time forever is not something that should be taken lightly. However, I like to understand how things work, so let's look under the hood and understand the basic functionality of a camera. 

In my estimation, there are a couple of main parts to focus on: 
1. The lens: Situated near the outside of the camera, the lens bends incoming light towards the sensor. 
2. The aperture: The opening in the camera lens, determining how much light enters it. Think of it as "a hole letting light in". 
3. Sensor: a small rectangle situated behind the lens, it contains many light receptors recording the intensity of light hitting the receptor. To capture colors, red green and blue filters are interchanged to get the intensity of each channel.  

Note that the sensor has some size, and that only light that hits the sensor will be recorded. This will be important later. Furthermore, note the distance between the lens and the sensor. This is called the *focal length*

We describe the point where "light meets" as the "focal point" of a camera. 

The idea is that by recording the magnitude and color of light hitting the sensor at each point, it gets a "map" of the scene. The reflection of light from a given object onto the sensor is what "records" that object in the photograph. If an object's light doesn't hit the sensor, it isn't in the photo. 

With that basic anatomy of a camera, now we can jump to our mathematical model. 

# Pinhole Camera Model
In math, when dealing with some physical system, we use a **model** to describe that system mathematically. Most often, this comes with a set of simplifications or assumptions which make our life easier, but hopefully still accurately describe the real-life phenomena. 

Here, we strip away much of the technical detail with our model, to use the simplest math we can. Namely, we take away the lens, and we narrow the aperature down to a SINGLE point (unfortunately impossible in the real world, although approximate pinhole cameras do exist). 

Thus, we have just a pinhole and a sensor. 

First, it's good to note how this simplifies things. To understand this, think of the light coming in through the aperture as a straw in a cup with no lid. You can put the strat at an angle and have it touch the center of the bottom of the cup. However, if you rotate it 180 degrees(or any amount) around the cup, you can still angle it so that it touches the center. What this means in the context of a camera, is that the same spot on the sensor can absorb light from 2 different sources. This is bad for our model, since we want the sensor to be like a "projection", and light coming in from different angles should represent distinct points. However, if we put a lid on the cup, and then tilt the straw around, each spot on the bottom corresponds to a SINGLE angle to the straw. This is what our pinhole does. 

Another simplification is that now without the lens, light isn't bent, so light hits the sensor at the angle it leaves our object. Thus, there isn't any "distortion". 


Now, to get to the model. 

Let $X$ be a point in 3-d space we wish to capture with the camera. 
Let the center of our camera (ie where the pinhole is located) be situated at the point $C$. 

We consider the camera being pointed in the +z direction, which we call the principal axis. Thus, the sensor is located on the plane (x, y, -f), where f is the focal length. 
However, this results in the image depicted on the camera being upside down and reversed. 

So instead, we use a "virtual image plane", placed at (x, y, f), for the model.

Thus, we consider the act of "taking a photograph" as a projection of 3d space onto this image plane. 

Doing geometry, if $$X = (x, y, z)$$, it projects onto the point $$(fX/Z, fY/Z, f)$$ on the virtual image plane. 



## Where am I? A study of coordinates

When you're handwavy about coordinate systems, it's easy to make subtle mistakes. Thus, it's time to be careful with them. 

First, we have a 3D coordinate system we denote the **world** coordinates. This can be visualized as  $$X, Y, Z$$ axes pointing in 3 orthogonal directions, situated at some origin O(and a bit more, as we'll clarify later). 

Note that these coordinates are NOT necessarily the same coordinates we're talking about when we said the Camera was pointed in the $+z$ direction. 

Thus, we will use the notation $$X_{W}$$ and $$X_{C}$$ to refer to a given point in the world and camera coordinate systems


Generally, our 3d point will be described in these world coordinates. However, when we project it, our calculation assumes camera coordinates. 

Thus, we need to know how to transform between the two coordinate systems. 

We define a matrix $$R_{cw} =\begin{bmatrix} r_x & r_y & r_z \end{bmatrix}$$, where $$r_x, r_y, and r_z$$ represent the coordinate axes of the Camera coordinate system written in terms
of the world coordinates, and we define a vector $$t_{cw}$$ to represent the origin of the camera coordinates in terms of the world frame. 

To transform from world coordinates to the camera coordinates, we do $$X_{C} = R_{cw}^T(X_{W} - t_{cw})$$
(If this formula seems backward, you're not alone. See my post about Change of Basis for the details here). 

Often, what we do instead is to let $$R = R_{wc} = R_{cw}^T$$, and $$t = t_{wc} = - R_{cw}^Tt_{cw}$$, and we say that $$X_{cam} = RX_w + t$$. 

I introduced it the other way to make clear what exactly we're doing with the rotation matrix. 

### Homogenous coordinates
A common convention used when discussing camera geometry is the use of **homogenous coordinates**, which lead to simpler expressions. 
This idea is rooted in Projective geometry and the Projective plane $RP_2$, but we won't go into that here. 

All you need to know is what they mean. Essentially, homogenous coordinates ADD another dimension to the current coordinates, then puts a value of "1" in that new dimension. 
So (X, Y, Z) becomes (X,Y, Z, 1).

What distinguishes this from a subset of $$\mathbb{R}^4$$ is that we say two vectors are equivalent if $$(X/W, Y/W, Z/W, W/W) = (x/w, y/w, z/w, w/w)$$
In other words, we say any rescaling of a vector is equivalent to that vector. The extra dimension just provides a nice way of describing our current "scaling" and translating between the two coordinate systems. 

This might seem unnecessary, but what it does is allows us to rewrite our previous formulas in a simpler manner. Instead of saying $$X_C = RX_w + t$$, we can represent both $$X_C$$ and $$X_W$$ in homogenous coordinates and say
$$X_W = \begin{bmatrix} R & t\\ 0 & 1 \\ \end{bmatrix} X_C$$. Now, our transformation is linear.

This coordinate system also gives us a simpler way of rewriting our earlier description of the projection onto the image plane: 
$$Z(fX/Z, fY/Z, f) = f\begin{bmatrix} I & 0 \end{bmatrix} X_C = f(X, Y, Z)$$. 

Here we've used homogenous coordinates to write the formula in a simpler manner, linearly. 

Thus, if we let $$x_i = fX/Z, y_i = fY/Z$$, we have the projection of our 3d coordinate onto the image plane, written in 2d coordinates. 

However, this isn't in the normal form we come to expect for images, which is **pixel** coordinates. Thus, we must do another linear transformation to go to those. 

To simplify calculations, we pull out the focal length coordinate from this step, and instead say that the homogenous coordinates of a point on the image plane is of the form $$(x_i, y_i, 1)$$
Then, to translate this to pixel coordinates, we scale by fx, fy in the x and y directions. In real life, $$f_x = f\cdot s_x$$, where f is the focal length in millimeters, and $$s_x$$ is the 
number of pixels represented per millimeter on the sensor. Then, we add $$c_x, c_y$$ to these values to offset the origin from the center of the image plane to the top left corner (where pixel coordinates originate). 

Note that we can write all of this using our newly developed homogenous coordinates, to say:

$$  X_p = K X_{im} = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}X_{im}$$, 

Where we write both $$X_p$$ and $$X_{im}$$ as homogenous 2d coordinates. Another word for the $$X_im$$ coordinate system (without mult by focal length) is **normalized image coordinates**, which are preferred for some algorithms. 

We call $$K$$ the **intrinsic** camera matrix. 


Now, we can write all of our transformation from world coordinates to pixel coordinates in one step. 

$$sx_p = PX_w = K \begin{bmatrix} I & 0 \end{bmatrix} \begin{bmatrix} R & t\\ 0 & 1 \\ \end{bmatrix}X_w = K\begin{bmatrix} R &  t \end{bmatrix}X_w$$$
We call the matrix $$\begin{bmatrix} R &  t \end{bmatrix}$$ the **extrinsic** parameter matrix, and P is the **camera** matrix. 

As you can see, after projection we lose information about how far away our point was from the origin. When reconstructing 3d coordinates from 2d ones, we'll need some way to obtain this depth 
parameter, either via a stereo setup or some external sensor. 

## Reverse projection

It's one thing to project one way, but how about the other? 
Well, immediately something becomes obvious: There is no well defined inverse to the camera matrix P, since it is 3x4. Thus, there is some information loss. 

We can visualize this by considering the picture of the camera center and the point on the pixel. If we draw a line between the two, and continue it out into space, we know that the 3d point the projected pixel 
came from must be on that line. However, we don't know the depth, since this was a projection. Thus, any point on that line would be a valid "inverse". 

Instead, we use the **Moore-Penrose** inverse. This is well defined on any size of matrix, and is essentially the "best inverse we can get", as in, $$\hat X = P^{\dagger}x$$ gives us the least squares guess of the true value. 
The system $PX = x$ is underdetermined, so the moore penrose gives us teh least square norm value. 

To visualize what the moore penrose inverse does, is it chooses the point on the line closest to the origin of the world coordinate system. As you can see, this is somewhat arbitrary. Thus, we need a better way. 

## Two camera setup
To make things somewhat more interesting, let's consider a stereo camera setup. We have two cameras with poses (R1, t1) and (R2, t2), and intrinsics $K_1, K_2$. Thus, we can take a point X and project it to BOTH cameras via their respective camera matrices, 
$$P_1 = K_1 @ \begin{bmatrix} R_1 & t_1 \end{bmatrix}$$ and likewise for $P_2$. 

However, when we take pictures with cameras, usually we don't have the 3D coordinate. Instead, we have two images at some related position, and we want to find corresponding points in each image. 

Well, if we already know the poses, it turns out that we can compute this without much difficulty. We only need the RELATIVE poses. 

## The fundamental matrix
The typical object of focus we aim to compute in the stereo camera setup is the fundamental matrix. 

What the fundamental matrix does is project a point on the first image to a line on the second image. 

Remember the "line" we get by reverse projecting a point on the first image? Well, if you project each point on that line onto the second image, you get another line (this time on the image plane). 
This other line, we call an **epipolar line**. Each point on the first image has a corresponding epipolar line on the second image. 

Now, I claim something which may seem surprising at first. I claim that all the epipolar lines intersect at ONE point. 
why would this be true? Well, if we're projecting EACH point on the ray between the camera center and a pixel onto the other image, one point which is on EVERY one of these rays is the camera center itself!

Therefore, the epipole $e2$ is the camera center of the first image projected onto the other image. Note that this may not be IN our image, which is ok. This represents the case where the neither camera would appear in the image of the other(if the other camera appeared in this image, its epipole would still be in there). 

You can use these epipoles in your code to test if things are really working. 




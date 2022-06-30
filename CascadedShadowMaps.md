# Cascading Shadow Maps

One of the main issues, that is faced during shadow mapping is there’s  only a limited range beyond which shadows don’t appear. So we firstly, try to fit the view frustum inside the frustum box from the perspective of the light. This can be done by first receiving the coordinates of the camera frustum in world space by multiplying the coordinates in NDC with the inverse of the view projection matrix. 

Now we can find the center of this frustum by simple math as well as find the max possible value of x,y,z and the min possible value, and using these values we can construct the best fitting box that covers the entire view frustum.



Now, we can render the shadow map along within this box region, and the shadow map changes as the camera is moved. So essentially, now we are able to see shadows anywhere.
But this doesn’t fix the problem of shadows in front of the camera. Basically, since the far plane of the view frustum is large, the shadows rendered are of pretty low quality(the smaller is the bounding box of the light’s view, the better quality the shadows are). So we deal with this problem by dividing the region between the near plane of the view frustum and the far plane of the view frustum into smaller frustums, and for each of these frustums we  create a shadow map, in a similar manner to the previous implementation. But in case of CSMs, we notice a huge dip in performance, which is due to high quality textures(4K resolution) being used. The performance issues can be somewhat minimized by reducing the size of the texture but that decreases the shadow quality.


``` cpp

std::vector<float> shadowCascadeLevels{ cameraFarPlane / 50.0f, cameraFarPlane / 25.0f, cameraFarPlane / 10.0f, cameraFarPlane / 2.0f };

constexpr unsigned int depthMapResolution = 4096;
```
![Alt text](4096.png?raw=true "Title")

This is how they look from a distance.

![Alt text](4096_far.png?raw=true "Title")

The transition between the two cascades is pretty smooth in this case

The first attempt to fix this was to do the obvious reduce the size of the depthMap.


``` cpp
constexpr unsigned int depthMapResolution = 4096/4;
```

![Alt text](4096by4.png?raw=true "Title")


As one can observe the shadow sharpness decreases, although it is less noticable.

The transition between cascades is way more observable, and the shadow quality gets much worse.

![Alt text](4096by4_far.png?raw=true "Title")

So we sacrifice performance for quality.

An attempt was made to fix this by introducing more cascade levels.

``` cpp
std::vector<float> shadowCascadeLevels{ cameraFarPlane / 100.0f, cameraFarPlane / 50.0f, cameraFarPlane / 25.0f, cameraFarPlane / 10.0f , cameraFarPlane / 5.0f , cameraFarPlane / 2.0f };
```

![Alt text](1024_moremaps.png?raw=true "Title")

![Alt text](1024moremaps_far.png?raw=true "Title")

The results were pretty satisfactory. While near, the maps look as good as the 4k ones despite using about 1/4th the resolution and more than double the performance.

While far maps transition look a bit worse, nevertheless it's much better than using just three shadow maps. They don't even look that bad!



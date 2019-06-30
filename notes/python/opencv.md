# OpenCV Notes

## General
* For historical reasons, OpenCV, by default, encodes color channels BGR instead of the more common RGB.  So, for example after reading an image into a numpy array with `cv2.imread()`, to convert an image to RGB use `cv2.cvtColor(input_image, flag)` where `flag` encodes the transformation, and is this case is `cv2.COLOR_BGR2RGB`.  Same function can be used to convert to grayscale (`cv2.COLOR_BGR2GRAY`) or several other color spaces.
* OpenCV represents images using a general purpose `Mat` class that is a matrix (and with an additional channels dimension of variable length).  Because matrix convention is (row, column), matrix operations follow this convention.  The `at(i,j)` function follows (row, column) convention.  This convention clashes with the Cartesian convention of (x,y) so spatial operations follows the opposite convention.  For example, operations in the camera calibration module follow (column, row) convention.  In both cases, the origin is in the top left corner.  [Source](https://stackoverflow.com/questions/25642532/opencv-pointx-y-represent-column-row-or-row-column)
* The `cv::Mat` class has two attributes that describe the datatype of the matrix.  The `depth` attribute describes the datatype of the individual elements of the matrix.  It is passed as an int with values defined by macros in `cvdef.h`.  Some possible values include CV\_16U, CV\_16S, CV\_32F, etc.  The `type` attribute combines depth with the number of channels.  Macros in `cvdef.h` literally do that.  An example would be CV\_16UC2 for an image where each pixel is stored as a 16 bit unsigned int with two channels. [Source](https://codeyarns.com/2015/08/27/depth-and-type-of-matrix-in-opencv/)

## Camera Calibration
* Optical distortion in a camera is related to _extrinsic_ and _intrinsic_ properties of the camera.
  * Intrinsic properties are those specific to and constant for a camera like focal lengths in X and Y dimensions and the optical center.  Together, these values are stored in a _camera matrix_ which can be reused for any images from that camera.
  * Extrinsic properties refer to the rotation and translation that maps a 3D point to an image coordinate.  Known rotation and translation values can be stored in a vector and can be resused for images taken from the same camera perspective.
* Knowledge of the intrinsic and extrinsic distortion properties can assist in determining the distortion function.  That function is itself composed of two portions:
  * _radial distortion_ can be described as a polynomial function of the radius from the _center of distortion_ which is usuallly the image center
  * _tangential distortion_ is caused by the plane of the image not being normal to the camera lense
* To compute the distortion function(s) coefficients (and extrinsic and intrinsic properties), OpenCV requires a series of images that have known 3D values and corresponding image points.  This is most easily accomplished with a shape like a chessboard that will have a set of points in a known plane and that can be easily discovered in an image.
* Useful OpenCV functions for this task include:
  * `cv2.findChessboardCorners()`takes an image and tuple describing number of interior corners and returns a array describing location of found corners
  * `cv2.drawChessboardCorners()` will mark the image with detected corners
  * `cv2.calibrateCamera()` uses the image points and known chessboard corner locations to calculate the distortion coeffcieints, camera matrix, and rotation/translation vectors
  * `cv2.undistort()` takes the values found by `calibrateCamera()` and applies transformations on the image to correct for distortion.

## Perspective Transformation
 * `warpPerspective()` applies a transformation, defined by a 3x3 matrix, to an input image.  Flags define interpolation methods used on borders and interior pixels.
 * `getPerspectiveTransform()` calculates the transformation matrix from 4 pairs of corresponding points.  The points are passed as a list of source points and a list of destination points.
 * The following example code tranforms an image of the roadway to an overhead view:

```python
def shift(img, direction):
    im_h, im_w = img.shape[0], img.shape[1]
    road_region = np.array(config.ROAD_REGION, dtype=np.float32)
    reg_w, reg_h = config.OVERHEAD_RECT
    rect = np.array([[im_w/2-reg_w/2, im_h],
                    [im_w/2-reg_w/2, im_h-reg_h],
                    [im_w/2+reg_w/2, im_h-reg_h],
                    [im_w/2+reg_w/2, im_h]],
                    dtype=np.float32)

    if direction:
        M = cv2.getPerspectiveTransform(road_region, rect)
    else:
        M = cv2.getPerspectiveTransform(rect, road_region)
    view = cv2.warpPerspective(img, M, (im_w, im_h))
    return view
```

## Sobel Operator
 * `Sobel()` calculates the image derivative using the Sobel operator applied to an input image.
   * `ksize` defines the size of the kernel.  If 1 is used, a 3x1 kernel is used that calculates the raw derivative approximation of adjacent pixels.  Larger `ksize` values apply 3x3, 5x5, or 7x7 kernels that include smoothing by a weighted averaging over adjacent rows/columns (Gaussian smoothing).

## Drawing Lines and Polynomials
 * `polylines()` draws polygonal curves on an input image.
   * `pts` is an _array_ of lines so to draw a single polyline, `pts=[[[x1,y1],[x2,y2],...]]`
   * color is a tuple of BGR values, `(blue, green, red)` from 0 to 255
 * `fillPoly()` draws solid, filled polygons on an image.  Like `polylines()`, it takes an array of polygons as input.
 * A partially transparent polygon can be drawn using a combination of `fillPoly()` and `addWeighted()`:
```python
# Create an image to draw the lines on
zero_channel = np.zeros_like(img).astype(np.uint8)
poly_img = np.dstack((zero_channel, zero_channel, zero_channel))

# Draw the lane onto the warped blank image
cv2.fillPoly(poly_img, np.int_([pts]), (0,255, 0)) # draws solid green polygon

# Combine the result with the original image
result = cv2.addWeighted(img, 1, poly_img, 0.3, 0) # overlays green polygon on image
```

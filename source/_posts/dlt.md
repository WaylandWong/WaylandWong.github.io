---
title: DLT 直接线性转换算法
r: true
s: dlt
date: 2022-10-26 19:02:11
tags:
- dlt
- computor vision
- algorithm
keywords:
- dlt
- computor vision
- algorithm
categories: ComputorVersion
description:
- DLT 直接线性转换算法
---
> Direct Linear Transformation


## 2D DLT算法
记某一点$X_i=(x_i,y_i,w_i)$，其映射后坐标$X'_i=(x'_i,y'_i, w'_i)$。
设2D点映射关系如下：
$$
X'_i=HX_i, \quad H= \begin{bmatrix}
h1 & h2 & h3 \\
h4 & h5 & h6 \\
h7 & h8 & h9
\end{bmatrix}
$$
$X'_i$和$HX_i$可能并不相等，它们有相同的方向但大小可能相差一个非零比例。由共线向量外积得零向量可转化为：
$$
\quad X'_i \times HX_i = 0
$$
记$h^T_j$为$H$矩阵的第$j$行，则有：
$$
H X_i = \left(
\begin{array} \\
h^T_1 X_i \\
h^T_2 X_i \\
h^T_3 X_i
\end{array}
\right)
$$
由向量[叉积 ](https://zh.wikipedia.org/wiki/%E5%8F%89%E7%A7%AF)d得：
$$
x'_i \times Hx_i = 
\left(
\begin{array} \\
x'_i \\
y'_i \\
w'_i
\end{array} 
\right) \times \left(
\begin{array} \\
h^T_1 X_i \\
h^T_2 X_i \\
h^T_3 X_i
\end{array} \right) = \left(
\begin{array} \\
y'_i h^T_3X_i - w'_i h^T_2X_i \\
w'_i h^T_1X_i - x'_i h^T_3X_i \\
x'_i h^T_2X_i - y'_i h^T_1X_i
\end{array} 
\right)   \tag{1}
$$

向量转为矩阵运算，以及矩阵转置得：

$$
(1) = \begin{bmatrix}
0 & -w'_i X_i & y'_i X_i \\
w'_i X_i & 0 & -x'_i X_i \\
-y'_i X_i & x'_i X_i & 0 \\
\end{bmatrix} 
\left(
\begin{array} \\
h^T_1 \\
h^T_2 \\
h^T_3
\end{array}
\right) = \begin{bmatrix}
0^T & -w'_i X^T_i & y'_i X^T_i \\
w'_i X^T_i & 0^T & -x'^T_i X^T_i \\
-y'_i X^T_i  & x'_i X^T_i & 0^T
\end{bmatrix}
\left(
\begin{array} \\
h_1 \\
h_2 \\
h_3
\end{array}
\right) = 0  \tag{2}
$$
可表示为：
$$
A_i H = 0
$$

$$
h= \left(
\begin{array} \\
h_1 \\
h_2 \\
h_3
\end{array}
\right)
$$
因为$X_i$是3维列向量，$H_1$是3维列向量，所以$A_i$是$3 \times 9$矩阵，$H$是9维列向量，
可以看到每一行的展开都会产生由3个方程组成的方程组，但每个方程中只有两个独立变量，所以可以忽略第三个方程，公式$(1)$可简化为：
$$
\begin{bmatrix}
0^T & -w'_i X^T_i & y'_i X^T_i \\
w'_i X^T_i & 0^T & -x'^T_i X^T_i
\end{bmatrix}
\left(
\begin{array} \\
h_1 \\
h_2 \\
h_3
\end{array}
\right) =0 \tag{3}
$$
此时$A_i$简化为$2 \times 9$矩阵。对于每一对点的映射关系可对$H$产生两个等式，给定4个等式可以得到一组方程：
$$
Ah=0, \quad AH=0
$$
其中$A$是由矩阵行$A_i$建立的方程系数，它的元素个数是已知点坐标的二次方。在图片处理中，一般选择$w_i=1$，也可以取其它数值。求$H$的非零解即可得到最终的结果。

在图像处理中，因为噪音或者误差的问题，我们会采集超过4个点来做处理，在这种情况下不是直接计算一个确定的值，而是寻找一个使得**损失函数**最小的近似的解。

在这种计算逻辑中，为了排除零解，可以约束$||h||=1$，它的值并不重要，因为$H$只是定义到了解的度。鉴于$Ah=0$没有确切的解，所以尝试取寻找$||Ah||$的最小值，也等同于寻找$||Ah||/||h||$的最小值。解就是$A^TA$的单位特征向量，它的特征值最小，参考[Multiple View Geometry in Computer Vision, Second Edition #P592 A5.3 Least-squares solution of homogeneous equations](https://www.changjiangcai.com/files/text-books/Richard_Hartley_Andrew_Zisserman-Multiple_View_Geometry_in_Computer_Vision-EN.pdf)，也即寻找矩阵$A$的最小奇异值的单位奇异向量，即可确定$X$和$X'$的映射关系。由此产生的算法被称为DLT算法。

初了直接将$H$作为齐次向量之外，还可以通过设置向量$h$某个分量$h_j=1$从而把方程组$(3)$ 变成非齐次矩阵。这里不再展开讨论，有兴趣可以参考[Multiple View Geometry in Computer Vision, Second Edition #P90 4.1.2 Inhomogeneous solution](https://www.changjiangcai.com/files/text-books/Richard_Hartley_Andrew_Zisserman-Multiple_View_Geometry_in_Computer_Vision-EN.pdf)。

问题：
- 对于4组映射点关系，矩阵A即可确定，直接求该矩阵的最小奇异值对应的单位奇异向量就是要求的线性映射关系H？奇异向量 -> 3x3矩阵？
- 如何对多个值（远超4组）求最小值，依次迭代吗？
- 相机内外参又如何映射到这个矩阵A里？
- 相机径向畸变会导致DLT误差增大

## 3D DLT 算法
从2D的$(3)$式转为3D场景下的公式：
$$
\begin{bmatrix}
0^T & -w'_i X^T_i & y'_i X^T_i \\
w'_i X^T_i & 0^T & -x'^T_i X^T_i
\end{bmatrix}
\left(
\begin{array} \\
p_1 \\
p_2 \\
p_3
\end{array}
\right) =0 \tag{3}
$$
$p$是$3 \times 4$的投影矩阵。

## DLT 3D 重建

当有了多个2维相机的线性映射之后，可以使用DLT来将2维坐标投影到3维空间中。下面应用简单的线性三角测量的方法。

对输入的每一张图片，有以下映射关系：
$$
x=PX, x' = P'X
$$
其中$X$是3D世界坐标系下的一个点，$x$是这个3D点在一个相机空间的坐标，$x'$是这个3D点另一个相机空间的坐标。这两个等式可以合并为$AX=0$，这个一个对变量$X$的线性方程。齐次比例因此可以通过叉乘计算消除，可以给出多台相机中的每个图像点给出三个方程。例如第一个图片中的一个点可以给出：
$$
x \times (PX) = 0
$$
展开后如下：
$$
x(p^T_3X) - (p^T_1X) = 0, \quad \\
x(p^T_3X) - (p^T_1X) = 0, \quad \\
x(p^T_3X) - (p^T_1X) = 0 
$$

对多个相机的方程合并后得到：$AX=0$
$$
A= \begin{bmatrix}
xp^T_3-p^T_1\\
yp^T_3-p^T_2 \\
x'p'^T_3-p'^T_1 \\
y'p'^T_3-p'^T_2
\end{bmatrix}
$$

使用SVD求解矩阵$A$，可以估计出$X$的值，从而可以估计任何点的三维坐标。其中的$p$即为相机的内外参矩阵组合而成。

## 相机内外参转为DLT矩阵
$$
P = \begin{bmatrix} 
f_x & 0 & c_x & 0 \\ 
0 & f_y & c_y & 0 \\
0 & 0 & 1 & 0
\end{bmatrix} \begin{bmatrix}
r_{11} & r_{12} & r_{13} & t_1 \\
r_{21} & r_{22} & r_{23} & t_1 \\
r_{31} & r_{32} & r_{33} & t_3 \\
0 & 0 & 0 & 1 
\end{bmatrix}, \quad
m=\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0
\end{bmatrix}
$$

$$
DLT = KmP, \quad
nDLT = DLT/DLT[3,4]
$$

## Code Implement

```python
# -*- coding: utf-8 -*-  
# file DLTx.py version .1  
"""  
I found this code at https://www.mail-archive.com/floatcanvas@mithis.com/msg00513.html  
  
Camera calibration and point reconstruction based on direct linear transformation (DLT).  
The fundamental problem here is to find a mathematical relationship between the  
 coordinates  of a 3D point and its projection onto the image plane. The DLT (a linear approximation to this problem) is derived from modelling the object and its projection on the image plane as a pinhole camera situation.In simplistic terms, using the pinhole camera model, it can be found by similar  
 triangles the following relation between the image coordinates (u,v) and the 3D point (X,Y,Z):   [ u ]   [ L1  L2  L3  L4 ] [ X ]   [ v ] = [ L5  L6  L7  L8 ] [ Y ]   [ 1 ]   [ L9 L10 L11 L12 ] [ Z ]                              [ 1 ]The matrix L is known as the camera matrix or camera projection matrix. For a  
 2D point (X,Y), the last column of the matrix doesn't exist. In fact, the L12 term (or L9 for 2D DLT) is not independent of the other parameters and then there are only 11 (or 8 for 2D DLT) independent parameters in the DLT to be determined.DLT is typically used in two steps: 1. camera calibration and 2. object (point)  
 reconstruction.The camera calibration step consists in digitizing points with known coordinates  
 in the real space.At least 4 points are necessary for the calibration of a plane (2D DLT) and at  
 least 6 points for the calibration of a volume (3D DLT). For the 2D DLT, at least one view of the object (points) must be entered. For the 3D DLT, at least 2 different views of the object (points) must be entered.These coordinates (from the object and image(s)) are inputted to the DLTcalib  
 algorithm which estimates the camera parameters (8 for 2D DLT and 11 for 3D DLT).With these camera parameters and with the camera(s) at the same position of the  
 calibration step,  we now can reconstruct the real position of any point inside the calibrated space (area for 2D DLT and volume for the 3D DLT) from the point position(s) viewed by the same fixed camera(s).This code can perform 2D or 3D DLT with any number of views (cameras).  
For 3D DLT, at least two views (cameras) are necessary.  
There are more accurate (but more complex) algorithms for camera calibration that  
 also consider lens distortion. For example, OpenCV and Tsai softwares have been ported to Python. However, DLT is classic, simple, and effective (fast) for most applications.About DLT, see: http://kwon3d.com/theory/dlt/dlt.html  
This code is based on different implementations and teaching material on DLT  
 found in the internet."""  
  
# Marcos Duarte - [EMAIL PROTECTED] - 04dec08  
  
import numpy as N  
import cv2  
import numpy as np  
import sys  
import os  
  
import cv2  
import json  
  
  
def getCameraNames(filename, node):  
    s = cv2.FileStorage(filename, cv2.FileStorage_READ)  
    data = s.getNode(node)  
    result = []  
    if not data.isSeq():  
        return result  
    for i in range(data.size()):  
        ele = data.at(i)  
        type = ele.type()  
        if type == 3:  
            # string  
            result.append(ele.string())  
        elif type == 1:  
            # int  
            result.append(str(int(ele.real())))  
    return result  
  
  
def getMat(filename, node):  
    s = cv2.FileStorage(filename, cv2.FileStorage_READ)  
    data = s.getNode(node).mat()  
    return data  
  
  
def getJoint(filename, jointType, index):  
    file = json.load(open(filename, 'r'))  
    joints = file['Joints'][jointType]  
    joint = joints[index*3: (index+1)*3 - 1]  
    return joint  
  
  
def IntriAndExtriToDLT(K, R, T):  
    # 相机内外参转dlt矩阵  
    # https://biomech.web.unc.edu/dlt-to-from-intrinsic-extrinsic/  
    # K = [ K 0    #       0 0]    # P = [R     T'    #     0 0 0 1]    P = np.append(R, T, axis=1)  
    P = np.append(P, [[0, 0, 0, 1]], axis=0)  
    zero = [[0, 0, 0]]  
    zero = N.transpose(zero)  
    K = np.append(K, zero, axis=1)  
    dlt = np.dot(K, P)  
  
    # normalize the result by dividing by the final value  
    ndlt = dlt / dlt[2, 3]  
    return ndlt.flatten()  
  
  
# 相机标定  
def DLTcalib(nd, xyz, uv):  
    '''  
    Camera calibration by DLT using known object points and their image points.    This code performs 2D or 3D DLT camera calibration with any number of views (cameras).    For 3D DLT, at least two views (cameras) are necessary.    Inputs:     nd is the number of dimensions of the object space: 3 for 3D DLT and 2 for 2D DLT.     xyz are the coordinates in the object 3D or 2D space of the calibration points.     uv are the coordinates in the image 2D space of these calibration points.     The coordinates (x,y,z and u,v) are given as columns and the different points as rows.     For the 2D DLT (object planar space), only the first 2 columns (x and y) are used.     There must be at least 6 calibration points for the 3D DLT and 4 for the 2D DLT.    Outputs:     L: array of the 8 or 11 parameters of the calibration matrix     err: error of the DLT (mean residual of the DLT transformation in units of camera coordinates).    '''  
    # Convert all variables to numpy array:  
    xyz = N.asarray(xyz)  
    uv = N.asarray(uv)  
    # number of points:  
    np = xyz.shape[0]  
    # Check the parameters:  
    if uv.shape[0] != np:  
        raise ValueError('xyz (%d points) and uv (%d points) have different number of points.' % (np, uv.shape[0]))  
    if (nd == 2 and xyz.shape[1] != 2) or (nd == 3 and xyz.shape[1] != 3):  
        raise ValueError('Incorrect number of coordinates (%d) for %dD DLT (it should be %d).' % (xyz.shape[1], nd, nd))  
    if nd == 3 and np < 6 or nd == 2 and np < 4:  
        raise ValueError(  
            '%dD DLT requires at least %d calibration points. Only %d points were entered.' % (nd, 2 * nd, np))  
  
    # Normalize the data to improve the DLT quality (DLT is dependent of the system of coordinates).  
    # This is relevant when there is a considerable perspective distortion.    # Normalization: mean position at origin and mean distance equals to 1 at each direction.    Txyz, xyzn = Normalization(nd, xyz)  
    Tuv, uvn = Normalization(2, uv)  
  
    A = []  
    if nd == 2:  # 2D DLT  
        for i in range(np):  
            x, y = xyzn[i, 0], xyzn[i, 1]  
            u, v = uvn[i, 0], uvn[i, 1]  
            A.append([x, y, 1, 0, 0, 0, -u * x, -u * y, -u])  
            A.append([0, 0, 0, x, y, 1, -v * x, -v * y, -v])  
    elif nd == 3:  # 3D DLT  
        for i in range(np):  
            x, y, z = xyzn[i, 0], xyzn[i, 1], xyzn[i, 2]  
            u, v = uvn[i, 0], uvn[i, 1]  
            A.append([x, y, z, 1, 0, 0, 0, 0, -u * x, -u * y, -u * z, -u])  
            A.append([0, 0, 0, 0, x, y, z, 1, -v * x, -v * y, -v * z, -v])  
  
    # convert A to array  
    A = N.asarray(A)  
    # Find the 11 (or 8 for 2D DLT) parameters:  
    U, S, Vh = N.linalg.svd(A)  
    # The parameters are in the last line of Vh and normalize them:  
    L = Vh[-1, :] / Vh[-1, -1]  
    # Camera projection matrix:  
    H = L.reshape(3, nd + 1)  
    # Denormalization:  
    H = N.dot(N.dot(N.linalg.pinv(Tuv), H), Txyz);  
    H = H / H[-1, -1]  
    L = H.flatten('C')  
    # Mean error of the DLT (mean residual of the DLT transformation in units of camera coordinates):  
    uv2 = N.dot(H, N.concatenate((xyz.T, N.ones((1, xyz.shape[0])))))  
    uv2 = uv2 / uv2[2, :]  
    # mean distance:  
    err = N.sqrt(N.mean(N.sum((uv2[0:2, :].T - uv) ** 2, 1)))  
  
    return L, err  
  
  
# 3D点投影回2D点  
def DLTProjection(Ls, xyz, uvs):  
    # m' = LM  
    size = N.shape(Ls)  
    p3d = N.transpose(xyz)  
    p3d = N.append(p3d, 1)  
  
    images = []  
    for i in range(size[0]):  
        K = Ls[i]  
        K = K.reshape(3, 4)  
        p = N.dot(K, p3d)  
        p = (p/p[2])  
        p = p[:2]  
        print("========↓↓↓↓↓↓↓projection↓↓↓↓↓↓↓=======")  
        print(i)  
        print(p)  
        # point to img and show  
        img = cv2.imread("quickpose/data/"+str(i)+"/0.jpg")  
        cv2.circle(img, (int(p[0]), int(p[1])), 5, (0, 0, 255), 4)  
        images.append(img)  
  
        print(uvs[i])  
        print("========↑↑↑↑↑↑↑original↑↑↑↑↑↑↑=========")  
  
    img = cv2.hconcat(images)  
    cv2.imshow("img-projection", img)  
    cv2.waitKey()  
  
  
def DLTrecon(nd, nc, Ls, uvs):  
    '''  
    Reconstruction of object point from image point(s) based on the DLT parameters.    This code performs 2D or 3D DLT point reconstruction with any number of views (cameras).    For 3D DLT, at least two views (cameras) are necessary.    Inputs:     nd is the number of dimensions of the object space: 3 for 3D DLT and 2 for 2D DLT.     nc is the number of cameras (views) used.     Ls (array type) are the camera calibration parameters of each camera      (is the output of DLTcalib function). The Ls parameters are given as columns  
      and the Ls for different cameras as rows.     uvs are the coordinates of the point in the image 2D space of each camera.      The coordinates of the point are given as columns and the different views as rows.    Outputs:     xyz: point coordinates in space    '''  
    # Convert Ls to array:  
    Ls = N.asarray(Ls)  
    # Check the parameters:  
    if Ls.ndim == 1 and nc != 1:  
        raise ValueError(  
            'Number of views (%d) and number of sets of camera calibration parameters (1) are different.' % (nc))  
    if Ls.ndim > 1 and nc != Ls.shape[0]:  
        raise ValueError(  
            'Number of views (%d) and number of sets of camera calibration parameters (%d) are different.' % (  
                nc, Ls.shape[0]))  
    if nd == 3 and Ls.ndim == 1:  
        raise ValueError('At least two sets of camera calibration parameters are needed for 3D point reconstruction.')  
  
    if nc == 1:  # 2D and 1 camera (view), the simplest (and fastest) case  
        # One could calculate inv(H) and input that to the code to speed up things if needed.        # (If there is only 1 camera, this transformation is all Floatcanvas2 might need)        Hinv = N.linalg.inv(Ls.reshape(3, 3))  
        # Point coordinates in space:  
        xyz = N.dot(Hinv, [uvs[0], uvs[1], 1])  
        xyz = xyz[0:2] / xyz[2]  
    else:  
        M = []  
        for i in range(nc):  
            L = Ls[i, :]  
            u, v = uvs[i][0], uvs[i][1]  # this indexing works for both list and numpy array  
            if nd == 2:  
                M.append([L[0] - u * L[6], L[1] - u * L[7], L[2] - u * L[8]])  
                M.append([L[3] - v * L[6], L[4] - v * L[7], L[5] - v * L[8]])  
            elif nd == 3:  
                M.append([L[0] - u * L[8], L[1] - u * L[9], L[2] - u * L[10], L[3] - u * L[11]])  
                M.append([L[4] - v * L[8], L[5] - v * L[9], L[6] - v * L[10], L[7] - v * L[11]])  
  
        # Find the xyz coordinates:  
        U, S, Vh = N.linalg.svd(N.asarray(M))  
        # Point coordinates in space:  
        xyz = Vh[-1, 0:-1] / Vh[-1, -1]  
  
    return xyz  
  
  
def Normalization(nd, x):  
    '''  
    Normalization of coordinates (centroid to the origin and mean distance of sqrt(2 or 3).    Inputs:     nd: number of dimensions (2 for 2D; 3 for 3D)     x: the data to be normalized (directions at different columns and points at rows)    Outputs:     Tr: the transformation matrix (translation plus scaling)     x: the transformed data    '''  
    x = N.asarray(x)  
    m, s = N.mean(x, 0), N.std(x)  
    if nd == 2:  
        Tr = N.array([[s, 0, m[0]], [0, s, m[1]], [0, 0, 1]])  
    else:  
        Tr = N.array([[s, 0, 0, m[0]], [0, s, 0, m[1]], [0, 0, s, m[2]], [0, 0, 0, 1]])  
  
    Tr = N.linalg.inv(Tr)  
    x = N.dot(Tr, N.concatenate((x.T, N.ones((1, x.shape[0])))))  
    x = x[0:nd, :].T  
  
    return Tr, x  
  
  
# 将所有相机的内外参转为DLT矩阵  
def convert():  
    names = getCameraNames("quickpose/intri.yaml", "names")  
    names.sort()  
    nd = 3  
    Ls = []  
    uvs = []  
    names = ["0", "1", "2", "3"]  
    nc = len(names)  
  
    images = []  
    for name in names:  
        K = getMat("quickpose/intri.yaml", "K_" + name)  
        T = getMat("quickpose/extri.yaml", "T_" + name)  
        vR = getMat("quickpose/extri.yaml", "R_" + name)  
        idx = 0  
        if name == "2":  
            idx = 3  
        if name == "3":  
            idx = 2  
        # if name == "4":  
        # 第四个人的数据不对  
        #     idx = 2  
        # 中间人的root节点  
        rootJoint = getJoint("quickpose/data/"+name+"/0.json", 1, idx)  
        # Converts a rotation matrix to a rotation vector or vice versa.  
        R = N.asarray(cv2.Rodrigues(vR)[0])  
        ndlt = IntriAndExtriToDLT(K=K, R=R, T=T)  
        if len(uvs) == 0:  
            uvs = rootJoint  
        else:  
            uvs = np.vstack((uvs, rootJoint))  
        if len(Ls) == 0:  
            Ls = ndlt  
        else:  
            Ls = np.vstack((Ls, ndlt))  
  
        img = cv2.imread("quickpose/data/"+name+"/0.jpg")  
        cv2.circle(img, (int(rootJoint[0]), int(rootJoint[1])), 5, (0, 0, 255), 5)  
        images.append(img)  
  
    img = cv2.hconcat(images)  
    cv2.imshow("img-origin", img)  
  
    xyz = DLTrecon(nd=nd, nc=nc, Ls=Ls, uvs=uvs)  
    print(xyz)  
    DLTProjection(Ls, xyz, uvs)  
    xyz1234 = N.zeros((len(xyz), 3))  
    print('Mean error of the point reconstruction using the DLT (error in cm):')  
    print(N.mean(N.sqrt(N.sum((N.array(xyz1234) - N.array(xyz)) ** 2, 1))))  
  
  
if __name__ == "__main__":  
    convert()  
  
# TODO: 1. 测试 IntriAndExtriToDLT 方法结果对不对

```


## 参考链接

- [叉积 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/%E5%8F%89%E7%A7%AF)
- [DLT Method (kwon3d.com)](http://kwon3d.com/theory/dlt/dlt.html)
- [DLT to/from intrinsic + extrinsic | Hedrick Lab :: Comparative Biomechanics (unc.edu)](https://biomech.web.unc.edu/dlt-to-from-intrinsic-extrinsic/)
- [3d-reconstruction-using-the-direct-linear-transform.pdf](http://bardsley.org.uk/wp-content/uploads/2007/02/3d-reconstruction-using-the-direct-linear-transform.pdf)
- [单应性Homograph估计：从传统算法到深度学习](https://zhuanlan.zhihu.com/p/74597564)
- [Multiple View Geometry in Computer Vision, Second Edition](https://www.changjiangcai.com/files/text-books/Richard_Hartley_Andrew_Zisserman-Multiple_View_Geometry_in_Computer_Vision-EN.pdf)
- [空间点的多视图DLT三维定位](https://www.opticsjournal.net/X_Article/GetArticlePDF/OJfc33ccbbb56b4016)
- [dlt python](https://www.mail-archive.com/floatcanvas@mithis.com/msg00513.html)
- [DLT to/from intrinsic + extrinsic | Hedrick Lab :: Comparative Biomechanics (unc.edu)](https://biomech.web.unc.edu/dlt-to-from-intrinsic-extrinsic/)

# Abaqus_Voronoi_toolbox

# Usage instructions

The developed toolbox supports ABAQUS versions ranging from 6.14 to 2025. The toolbox is compatible with both Python 2.x and Python 3.x environments, as ABAQUS/CAE employs Python 2.x for versions prior to 2022 and Python 3.x from version 2022 onward. A key advantage of the developed toolbox is its seamless integration into the standard ABAQUS/CAE workflow as a plug-in. The generation of polycrystalline microstructures becomes therefore straightforward and can be summarized in three main steps:

• Create the model geometry within the ABAQUS ‘Part’ module, or import it from external CAD software.

• Discretize the geometry using the ABAQUS ‘Mesh’ module, or import the mesh file from external software (if the geometry is already discretized, Step 1 can be skipped).

• Launch the ABAQUS-Voronoi toolbox from the Plug-ins menu (see the following figure), specify the required parameters, and generate the Voronoi microstructure.

<img width="1244" height="208" alt="2024-09-21_173359" src="https://github.com/user-attachments/assets/628b2708-8bfc-4b0b-8a53-1aa649a78c7a" />


# Compare with NEPER software
## The RVE model without void

NEPER command: 

• neper -T -n 200 -domain 'cube(1,1,1)' -morphooptiini "coo:file(random_points.txt)" -o ccc  #random_points.txt in the download folder

• neper -M ccc.tess -elttype hex -rcl 0.91

• neper -V ccc.msh -dataelsetcol id -showelset "(id==1)||(id==120)" -cameraangle 15 -imagesize 1200:800 -print image_xxc

• neper -V ccc.tess,ccc.msh -dataelsetcol id -imagesize 1200:800 -print image_18

<img width="1255" height="960" alt="ScreenShot_2026-04-16_170931_628" src="https://github.com/user-attachments/assets/e0723aa7-3803-4c23-a99b-2efe45ffde94" />

## The RVE model with void
<img width="1010" height="459" alt="ScreenShot_2026-04-17_153233_512" src="https://github.com/user-attachments/assets/efd110ea-2cef-45ba-a0c1-f5a727ec5a2f" />



# Advanced RVEs generated using the developed toolbox


## 1 RVE with a gradient structure
Gradient-structured metals represent an emerging class of advanced materials, characterized by outstanding combination of strength and ductility (Li et al., 2017; Zhao et al., 2021; Liu et al., 2024). Accurately modeling such functionally graded microstructures remains a significant challenge, for which the user-defined module of the developed toolbox is particularly suitable. The following figure illustrates an example of a gradient-structured RVE, with fine grains at the top transitioning to coarse grains at the bottom. The generation of this complex RVE can be achieved through a straightforward two-step workflow:

• Generate gradient seeds: define a set of seed coordinates with a prescribed spatial distribution. The Python script below demonstrates how to vary the seed density along the \mathit{\mathrm{Z}}-axis to produce the desired gradient.
• Import seeds into the toolbox: the generated seed coordinates are imported into the user-defined module. After verifying all inputs within the module, the final RVE is generated.


```python
import numpy as np
np.random.seed(42)
num_points = 1000 #the number of seeds
B = np.zeros((num_points, 3))
B[:, 0] = np.random.rand(num_points)
B1 = np.abs(np.random.randn(num_points))
B[:, 2] = 1 - B1 / np.max(B1)
B[:, 1] = np.random.rand(num_points)
np.savetxt('points.txt', B)
```

<img width="944" height="859" alt="A1" src="https://github.com/user-attachments/assets/6b8e9d58-b27e-40b2-b90c-dcbf0e9e7921" />

## 2 RVE with a rough surface
Surface quality and roughness are critical factors influencing component performance, particularly in localized necking predictions (Liu et al., 2020; Bong and Lee, 2021; Liu et al., 2022). Modeling these features is essential, but remains a major challenge for conventional RVE generators, which create the mesh and grain structure simultaneously. The following figure shows a polycrystalline RVE with a rough top surface. The generation of this complex RVE has been carried out in two steps:

• The open-source ABAQUS plug-in RufGen (Lim and Ha, 2023) has been used to generate a finite element model with a controlled rough surface. Here, the surface roughness parameters are set to: Lx=Ly=Lz=1, Le=0.03, Nx=Ny=100, and std=0.005.

• The general module of the developed toolbox has been applied directly to the pre-meshed rough model to partition it into 200 grains.

This example further highlights the flexibility and reliability of the proposed toolbox in partitioning the pre-existing meshes with complex geometric features.


<img width="915" height="887" alt="A2" src="https://github.com/user-attachments/assets/78b5ddee-dabf-4aa7-a5e9-11672ee603de" />

## 3 Cylindrical RVE

Cubic RVEs are commonly used, while cylindrical RVEs are also widely employed in various engineering simulations Tang et al. (2023). Generating polycrystalline microstructures within such non-cubic geometries remains a challenge for many RVE generation tools. This example demonstrates how the user-defined module of the developed toolbox can efficiently handle this task. The following figure illustrates a cylindrical RVE composed of 200 grains. The generation procedure involves three steps:

• The desired cylindrical part is created and meshed using standard tools within ABAQUS/CAE.

• A custom script is used to generate seed coordinates, randomly distributed within the cylindrical volume. A Python template for this procedure is provided below.

```python
import numpy as np
np.random.seed(42)
radius = 5 #the radius of the cylinder      
height = 20 #the height of the cylinder     
num_points = 200 #the number of seeds
center = (5, 5) #the center of the cylinder
points = []
while len(points) < num_points:
    x = np.random.uniform(center[0] - radius, center[0] + radius)
    y = np.random.uniform(center[1] - radius, center[1] + radius)  
    if (x-center[0])**2 + (y-center[1])**2 <= radius**2:
        z = np.random.uniform(0, height)
        points.append([x, y, z])
points = np.array(points)
np.savetxt('cylinder_points.txt', points)
```
• The file containing the seed coordinates is imported into the user-defined module, which applies the Voronoi tessellation directly onto the pre-existing cylindrical mesh.



<img width="758" height="692" alt="A3-1" src="https://github.com/user-attachments/assets/086b3e38-5b4e-430f-aa4d-4e16446be249" />

# Reference

Li J, Weng GJ, Chen S, Wu X (2017) On strain hardening mechanism in gradient nanostructures. Int J Plast. 88:89–107. https://doi.org/10.1016/j.ijplas.2016.10.003

Zhao F, Xie J, Zhu Y, Liu Q (2021) A novel dynamic technique to form reverse grain size gradient microstructure in pure copper. Mater Lett. 291:129581. https://doi.org/10.1016/j.matlet.2021.129581

Liu CZ, Xiong QL, Wen A (2024) Shear localization in gradient high-entropy alloy at high strain rates: Crystal plasticity modeling. Extrem Mech Lett. 70:102194. https://doi.org/10.1016/j.eml.2024.102194

Liu C, Xu W, Liu M, Zhang L (2020) Effect of strain path on roughness evolution of free surface during plastic deformation. Int J Mech Sci. 173. https://doi.org/10.1016/j.ijmecsci.2020.105475

Bong HJ, Lee J (2021) Crystal plasticity finite element-Marciniak-Kuczynski approach with surface roughening effect in predicting formability of ultra-thin ferritic stainless steel sheets. Int J Mech Sci. 191:106066. https://doi.org/10.1016/j.ijmecsci.2020.106066

Liu C, Xu W, Niu T, Chen Y (2022) Roughness evolution of constrained surface based on crystal plasticity finite element model and coupled Eulerian-Lagrangian method. Comput Mater Sci. 201:110900. https://doi.org/10.1016/j.commatsci.2021.110900

Lim Y, Ha S (2023) RufGen: A plug-in for rough surface generation in Abaqus/CAE. SoftwareX. 22:101380. https://doi.org/10.1016/j.softx.2023.101380

Tang X, Wang Z, Wang X, Deng L, Zhang M, Gong P, et al. (2023) Unraveling size-affected plastic heterogeneity and asymmetry during micro-scaled deformation of CP-Ti by non-local crystal plasticity modeling. Int J Plast. 170. https://doi.org/10.1016/j.ijplas.2023.103733

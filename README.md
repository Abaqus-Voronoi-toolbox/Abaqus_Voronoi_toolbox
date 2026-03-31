# Abaqus_Voronoi_toolbox

# Usage instructions

A key advantage of the developed toolbox is its seamless integration into the standard ABAQUS/CAE workflow as a plug-in. The generation of polycrystalline microstructures becomes therefore straightforward and can be summarized in three main steps:

• Create the model geometry within the ABAQUS ‘Part’ module, or import it from external CAD software.

• Discretize the geometry using the ABAQUS ‘Mesh’ module, or import the mesh file from external software (if the geometry is already discretized, Step 1 can be skipped).

• Launch the ABAQUS-Voronoi toolbox from the Plug-ins menu (see Fig. [fig:C.6]), specify the required parameters, and generate the Voronoi microstructure.

<img width="1244" height="208" alt="2024-09-21_173359" src="https://github.com/user-attachments/assets/628b2708-8bfc-4b0b-8a53-1aa649a78c7a" />


# Advanced RVEs generated using the developed toolbox


## 1 RVE with a gradient structure
Gradient-structured metals represent an emerging class of advanced materials, characterized by outstanding combination of strength and ductility (Li et al., 2017; Zhao et al., 2021; Liu et al., 2024). Accurately modeling such functionally graded microstructures remains a significant challenge, for which the user-defined module of the developed toolbox is particularly suitable. Figure [fig:A1] illustrates an example of a gradient-structured RVE, with fine grains at the top transitioning to coarse grains at the bottom. The generation of this complex RVE can be achieved through a straightforward two-step workflow:

 • Generate gradient seeds: define a set of seed coordinates with a prescribed spatial distribution. The Python script below demonstrates how to vary the seed density along the \mathit{\mathrm{Z}}-axis to produce the desired gradient.
• Import seeds into the toolbox: the generated seed coordinates are imported into the user-defined module (see Section [subsec:C.2.3]). After verifying all inputs within the module, the final RVE is generated.



import numpy as np
np.random.seed(42)
num_points = 1000 #the number of seeds
B = np.zeros((num_points, 3))
B[:, 0] = np.random.rand(num_points)
B1 = np.abs(np.random.randn(num_points))
B[:, 2] = 1 - B1 / np.max(B1)
B[:, 1] = np.random.rand(num_points)
np.savetxt('points.txt', B)

<img width="944" height="859" alt="A1" src="https://github.com/user-attachments/assets/6b8e9d58-b27e-40b2-b90c-dcbf0e9e7921" />

## 2 RVE with a rough surface
Surface quality and roughness are critical factors influencing component performance, particularly in localized necking predictions (Liu et al., 2020; Bong and Lee, 2021; Liu et al., 2022). Modeling these features is essential, but remains a major challenge for conventional RVE generators, which create the mesh and grain structure simultaneously. Fig. [fig:A2] shows a polycrystalline RVE with a rough top surface. The generation of this complex RVE has been carried out in two steps:

• The open-source ABAQUS plug-in RufGen (Lim and Ha, 2023) has been used to generate a finite element model with a controlled rough surface. Here, the surface roughness parameters are set to: Lx=Ly=Lz=1, Le=0.03, Nx=Ny=100, and std=0.005.

• The general module of the developed toolbox has been applied directly to the pre-meshed rough model to partition it into 200 grains.

This example further highlights the flexibility and reliability of the proposed toolbox in partitioning the pre-existing meshes with complex geometric features.



## 3 Cylindrical RVE

Cubic RVEs are commonly used, while cylindrical RVEs are also widely employed in various engineering simulations Tang et al. (2023). Generating polycrystalline microstructures within such non-cubic geometries remains a challenge for many RVE generation tools. This example demonstrates how the user-defined module of the developed toolbox can efficiently handle this task. Fig. [fig:A3] illustrates a cylindrical RVE composed of 200 grains. The generation procedure involves three steps:

• The desired cylindrical part is created and meshed using standard tools within ABAQUS/CAE.

• A custom script is used to generate seed coordinates, randomly distributed within the cylindrical volume. A Python template for this procedure is provided below.
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
• The file containing the seed coordinates is imported into the user-defined module, which applies the Voronoi tessellation directly onto the pre-existing cylindrical mesh.


<img width="758" height="692" alt="A3-1" src="https://github.com/user-attachments/assets/086b3e38-5b4e-430f-aa4d-4e16446be249" />


![logo](http://libgdx.badlogicgames.com/img/logo.png)
![logo](http://download.blender.org/institute/logos/blender-plain.png)

Blender G3D Exporter
====================

This is an addon for the Blender 3D modeling tool (http://www.blender.org). The purpose of this addon is to allow Blender to export 3D models as G3DJ files (G3D models written as JSON objects).

G3D is a custom 3D model file format compatible with the LibGDX framework (http://http://libgdx.badlogicgames.com/). The format is easy enough to allow use for other frameworks and is powerfull enough to support most features for game development, like multiple materials, multiple textures, different kinds of textures (diffuse maps, normal maps, etc.) and even armature based animations. The specification for this format can be found at [the project's wiki page](https://github.com/libgdx/fbx-conv/wiki) (work in progress).

This addon is currently compatible with Blender v2.76. I can't garantee compatibility with previous or future versions, but I'll try to update it if future Blender updates break compatibility.

This software is licensed under the [GNU General Public License Version 3.0](http://www.gnu.org/licenses/gpl-3.0.txt) (see LICENSE).

### Installation

You have to install this script as a Blender add-on. To do this copy the *io_scene_g3d* folder into the Blender *../scripts/addons* folder. These are the most common locations for this folder:

* **Windows 7 / 8** - C:\\Users\\%username%\\AppData\\Roaming\\Blender Foundation\\Blender\\_$version_\\scripts\\addons 
* **Windows XP** - C:\\Documents and Settings\\%username%\\Application Data\\Blender Foundation\\Blender\\_$version_\\scripts\\addons 
* **Linux** - /home/_$user_/.config/blender/_$version_/scripts/addons
* **All** (when addons are inside the Blender folder) - *$blender_folder*/*$version*/scripts/addons

After copying the folder to the correct place, open Blender and go to the *File -> User Preferences* menu under the Addons tab. Select the *Import-Export* category and enable the *LibGDX G3D Exporter* addon.

Usually you have to activate the addon for each new file. To have it already enabled for all new files first go to *File -> New* to reset your workspace (remember to save your work), enable the addon and then go to *File -> Save Startup File* to make this your default new file.

### Usage

This addon will add two new submenus in Blender under *File -> Export*. The option *LibGDX G3D text format* will export files to LibGDX JSON format (\*.g3dj) while the option *LibGDX G3D binary format* will export files to LibGDX binary format (\*.g3db).

These are the current options you can use to configure the exporting process.

* **Forward** and **Up** - Bleder uses a Z-up coordinate system while LibGDX by default uses a Y-up coordinate system. Here you can configure what to use Blender "Y" (forward) coordinate for and what to use Blender "Z" (up) coordinate for. The default is use Blender "Y" as LibGDX "-Z" and Blender "Z" as LibGDX "Y". Both Blender and LibGDX uses a right-handed coordinate system so whatever axis left will always point right. Ex: if you select "Z" as forward and "Y" as up then the exporter will user "negative X" as right.
* **Selection Only** - Only export selected objects and armatures. The exporter is only able to export mesh objects and armatures; lights, cameras and other kind of objects are always ignored. Default is *unchecked*.
* **Export Armatures** - Uncheck to ignore armatures even if *Selection Only* is unchecked. Default is *checked*.
* **Bone Weights per Vertex** - Defines the maximum number of bones a vertex can be weighted to, after which bones will be just ignored. The default for both the exporter and LibGDX basic shader is 4, since for a triangle (three vertices) it will make a maximum of 12 bones per face which is a safe number for most devices.   
* **Export Actions as Animations** - Check to export actions as bone animations. Only used actions are exported, if you have several actions for a single armature use the *FAKE USER* option in Blender to make all actions *used* (have at least one user). Default is *checked*.
* **Export Tangent and Binormal Vectors** - Calculate tangent and binormal vectors for each vertex (used for Normal Mapping). You need to create an UV map for your model. The tangent and binormal will be exported as the *TANGENT* and *BINORMAL* vertex attributes. Default is *unchecked*.

### Features

These are some of the things that get exported by this script:

* Multiple materials for the same mesh (will create multiple mesh parts)
* Diffuse, specular, opacity and shininess material attributes
* Multiple UV coordinates for texture mapping
* Various kinds of texture mappings (diffuse, normal, transparency, specular)
* Armatures
* Multiple bone animations

### Tips and tricks

These are the things to keep in mind when exporting to ensure your model gets exported correctly:


**If your Blender mesh object has too many bones consider splitting it into different parts** - The default shader in LibGDX supports 12 bones per face. This limit can be increased in code (setting the `DefaultShader.Config.numBones` property) but if some of your faces are weighted to more than 12 bones (surpassing the default of 4 per vertex) consider splitting your mesh into multiple parts where each part's faces have at most 12 bones.

To do that you can use multiple objects or you can use multiple materials. When using multiple materials, use the "Assign" button to assign each material to part of your mesh (select the vertices that make the part and hit "assign" in the Material panel), the exporter will create a mesh part for each group of vertices assigned to a different material. 

The reason for this is that a bone is nothing more than a 4x4 matrix, meaning that each bone is composed of 16 floating point values giving a total of 192 floats for 12 bones. On a GPU there is a limited number of floats that can be stored per render pass - it varies depending of the GPU but to give an example the PowerVR SGX supports 512 floats on the vertex shader. Bottom line, try to keep the number of bones per object low. More info on the matter [can be found here](http://www.badlogicgames.com/forum/viewtopic.php?f=11&t=12910).

**Exporting normal maps** - Normal maps are textures that encode a map of normal vectors. They are used by shaders that simulate a larger level of detail than the model really has. To export normal maps correctly you need to follow the steps below:

* **Generate the normal map** - There are tutorials on how to generate normal maps with Blender.
* **Create an UV map** for the mesh that will receive the texture.
* **On the texture panel** you need to check *Normal Map* under the Image Sampling subpanel and *Normal* under the Influence panel. If you only check the option under the Influence panel a "BUMP" type of texture will be exported instead.

**Exporting animations** - First of all, the exporter will use Actions (created using the Action Editor) to export animations. Only curves assigned to bones will be exported.

LibGDX uses a linear interpolator by default to animate bones. If you use any other algorithm to interpolate curves then the exporter will generate extra keyframes to compensate for that. To avoid that set your curves to linear interpolation, that way only the keyframes you create will be exporter.

Remember to set the animation framerate and the range of the animation on the Render panel. The exporter will use these settings to calculate the keyframe times.

### About the license

The intention was to make this script licensed under the Apache License v2 (same as LibGDX) until I found out any Python scripts that use the Blender API need to be licensed under the GNU General Public License. The site says they're looking into a way to allow other licenses but until that this software will be licensed under GNU General Public License v3.

Any *.g3dj/.g3db* files exported by this script are considered program output and as such are copyrighted to the user. You are free to use them as you see fit.

**This addon uses Simple UBJSON module for Python**. Simple UBJSON is licensed under the BSD license, copyright information can be found in the LICENSE file contained inside the *simpleubjson* package. The source code was modified to generate binary JSON files that can be read by LibGDX UBJSON reader, this source code modification is permited by the author through the BSD license.

### Changes

* **0.2.0**
  - Overall refactoring of the addon. New options should be easier to implement from now on.
  - The exporter now supports both text and binary G3D formats.
  - Added option to convert coordinate systems.
  - Adjusted material exporting to match the ones generated by the FBX-CONV tool.
  - The exporter no longer creates a default material for objects missing one, now a warning message appears when such case occurs.
  - Added option to limit the number of bones per vertex, respecting LibGDX default of 4.
  
* **0.1.0**
  - First release
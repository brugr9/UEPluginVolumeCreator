# Volume Creator: An Unreal&reg; Engine Plugin for Medical Data Rendering &ndash; Readme

This document is part of *"Volume Creator: An Unreal&reg; Engine Plugin for Medical Data Rendering &mdash; Documentation"*

* Author: Copyright 2023 Roland Bruggmann aka brugr9
* Profile on UE Marketplace: [https://www.unrealengine.com/marketplace/profile/brugr9](https://www.unrealengine.com/marketplace/profile/brugr9)
* Profile on Epic Developer Community: [https://dev.epicgames.com/community/profile/PQBq/brugr9](https://dev.epicgames.com/community/profile/PQBq/brugr9)

---

<!-- UE Marketplace : Begin 1/2 -->

![Featured Image](Docs/FeaturedImage894x488.png "Featured Image")

Adds Blueprint Support for Real-time Rendering from DICOM&reg; based Medical Imaging Data.

## Description

Unreal&reg; Engine plugin "Volume Creator" enables real-time multiplanar and direct volume rendering from the Blueprint Visual Scripting system.

The delivered assets provide importing DICOM&reg; based medical imaging data, applying DICOM Window and rendering by coloring from look-up tables and color-gradient based transferfunctions. With a clipping plane and/or with a region of interest the user may shrink the rendered volume interactively. The plugin allows to create solutions that can be used in VR/AR serious games, e.g., for teaching and training in medical education.

<!-- UE Marketplace : End 1/2 -->

* Index Terms:
  * Medical Imaging, Computer Tomography, Magnetic Resonance
  * Multiplanar Rendering, Direct Volume Rendering
  * Virtual Reality, Augmented Reality, Serious Games for Teaching and Training
* Technology: DICOM, Unreal Engine C++ Code Plugin, HLSL Compute Shader

* Tags: DICOM, CT, MR, MPR, DVR, VR, AR, UE, CS

---

<div style='page-break-after: always'></div>

## Table of Contents

<!-- Start Document Outline -->

* [1. Setup](#1-setup)
  * [1.1. Installation](#11-installation)
  * [1.2. Project Configuration](#12-project-configuration)
* [2. Concept](#2-concept)
  * [2.1. Scalar Volume](#21-scalar-volume)
  * [2.2. Region of Interest](#22-region-of-interest)
    * [2.2.1. ROI-SV](#221-roi-sv)
    * [2.2.2. ROI-MPR-3D](#222-roi-mpr-3d)
    * [2.2.3. ROI-Handles](#223-roi-handles)
  * [2.3. Clip Plane](#23-clip-plane)
  * [2.4. Spot-Light](#24-spot-light)
  * [2.5. Workflows](#25-workflows)
* [3. Blueprint SV and Inheriting Actors](#3-blueprint-sv-and-inheriting-actors)
  * [3.1. Actor BP SV H](#31-actor-bp-sv-h)
    * [3.1.2. Dataset](#312-dataset)
    * [3.1.3. DICOM Window](#313-dicom-window)
    * [3.1.4. Volume Rendering](#314-volume-rendering)
    * [3.1.5. Volume Shading](#315-volume-shading)
  * [3.2. Actor BP SV W](#32-actor-bp-sv-w)
  * [3.3. Actor BP SV W L](#33-actor-bp-sv-w-l)
* [4. Import](#4-import)
  * [4.1. Import DICOM](#41-import-dicom)
  * [4.2. Import MetaImage](#42-import-metaimage)
  * [4.3. Data Background](#43-data-background)
    * [4.3.1. Memory](#431-memory)
    * [4.3.2. Processing](#432-processing)
* [6. Outlook](#6-outlook)
* [Appendix](#appendix)
  * [Abbreviations and Acronyms](#abbreviations-and-acronyms)
  * [Glossary](#glossary)
    * [Terms of Location and Coordinate Systems](#terms-of-location-and-coordinate-systems)
    * [Asset Naming Convention](#asset-naming-convention)
  * [A. References](#a-references)
    * [A.1. Medical Imaging](#a1-medical-imaging)
    * [A.2. Unreal Engine](#a2-unreal-engine)
  * [B. Readings](#b-readings)
  * [C. Acknowledgements](#c-acknowledgements)
  * [D. Attribution](#d-attribution)
  * [E. Disclaimer](#e-disclaimer)
  * [F. Citation](#f-citation)

<!-- End Document Outline -->

<div style='page-break-after: always'></div>

## 1. Setup

### 1.1. Installation

In the Unreal Editor access the Plugin Editor from the menu 'Edit > Plugins'. In the Plugin Editor, under category 'Rendering' find and enable the plugin. Finally restart the Unreal Editor.

![Screenshot of Plugin Editor with Plugin 'Volume Creator' enabled](Docs/ScreenshotPlugin.jpg "Screenshot of Plugin Editor with Plugin 'Volume Creator' enabled")
<br>*Fig. 1.1.: Screenshot of Plugin Editor with Plugin "Volume Creator" enabled*

### 1.2. Project Configuration

To allow Volume Texture asset creation follow these steps as from Unreal Engine Documentation article [*Creating Volume Textures*](https://docs.unrealengine.com/4.26/en-US/RenderingAndGraphics/Textures/VolumeTextures/CreatingVolumeTextures/):

> Before you can use Volume Textures in your Unreal Engine 4 (UE4) project, you will need to enable them. In the following How-To, we will take a look at setting up your UE4 project to use Volume Textures.
>
> 1. First, make sure that the Editor is closed, and then locate your project's DefaultEngine.ini file and open it.
> 2. Locate the Script/Engine.RendererSettings section and add the following variable, then save the file when you have added it:
>
> ```r.AllowVolumeTextureAssetCreation=1```
>
> 3. Re-launch the Editor

<div style='page-break-after: always'></div>

## 2. Concept

The following entities are implemented as an object or Actor resp. according to the Object Oriented Paradigm OOP:

* **Scalar Volume SV**: The plugin provides rendering of image-stack based volumes, commonly known as scalar volumes *SV*. The plugin however does not support the rendering of other type of volumes, like vector volumes or tensor volumes. The scalar volume datasets are handled in Blueprint Actors &mdash; which includes handling, e.g., DICOM pixel spacing attribute values.
* **Multi-Planar Rendering MPR**: A scalar volume SV may be rendered as Multi-Planar Rendering, available in two versions: as **MPR-2D** and as **MPR-3D**.
  * The MPR-3D extents are shown as three perpendicular planes, i.e. coronal COR, sagittal SAG and axial AXE plane.
  * The MPR-3D planes position can be moved in the direction of their corresponding axes interactively in real-time.
  * The MPR-3D produces planar rendering as Render Target Texture RT.
  * The RT are also consumed by the MPR-2D.
* **Direct Volume Rendering DVR**: A scalar volume may be rendered as Direct Volume Rendering DVR.
  * **Bounding Box BB**: The DVR extents are shown with a bounding box BB, its dimension derives from the scalar volume DICOM pixel spacing attribute values.
  * **Static Spot-Light**: The DVR can be optionally illuminated with static spot-lights.
  * **Clip Plane**: The DVR geometry can be optionally shrinked with a clip plane interactively in real-time.
  * **Region of Interest ROI**: The DVR geometry can be optionally shrinked with a region of interest interactively in real-time.
    * **ROI-Handles**: A ROI geometry can be optionally modified with region of interest handles interactively in real-time. The ROI-Handels can

TODO: Domain Model Diagram

### 2.1. Scalar Volume

In this plugin, image-stack based scalar volumes are kept in `VolumeTexture` assets (suffix `_Volume`). A `VolumeTexture` may serve as voxel container of three different kind of content. The assets name suffix indicates the intended content type (cp. section *[Asset Naming Convention](#7-asset-naming-convention)*):

* **_HU_Volume**: CT image stack data, values in the range of [-1000, 3095] in Hounsfield Units
* **_W_Volume**: DICOM Window applied data, values in the range of [0, 255] Grayscale
* **_L_Volume**: Lightmap from static lighting

Plugin Content:

* Volume Textures (Default Containers): `T_SV_HU_Volume`, `T_SV_W_Volume`, `T_SV_L_Volume`
* Volume Render Textures (Shader Textures): `RT_SV_W_Volume`, `RT_SV_L_Volume`

TODO: Multiplanar Rendering MPR or Direct Volume Rendering DVR

<div style='page-break-after: always'></div>

### 2.2. Region of Interest

The plugin provides with a Blueprint actor named `BP_ROI` with a `StaticMeshComponent` of type `Cube`. The material instance `MI_FramingEdges_ROI` is assigned to the mesh (see figure 2.2.).

Plugin Content:

* Blueprint, Abstract Actor: `BP_ROI`
* Blueprint, Interface: `BPI_ROI`

![Blueprint Actor BP_ROI](Docs/BP_ROI.png "Blueprint Actor BP_ROI")<br>*Fig. 2.2.: Blueprint Actor BP_ROI*

#### 2.2.1. ROI-SV

* Material Instance: `MI_Edges_ROI`

#### 2.2.2. ROI-MPR-3D

* Material Instance: `MI_ROI`

#### 2.2.3. ROI-Handles

### 2.3. Clip Plane

The plugin provides with a Blueprint actor named `BP_ClipPlane` with a `StaticMeshComponent` of type `Plane`. The material instance `MI_FramingEdges_ClipPlane` is assigned to the mesh (see figure 2.3.).

Plugin Content:

* Blueprint Actor: `BP_ClipPlane`
* Material Instance: `MI_FramingEdges_ClipPlane`

![Blueprint Actor BP_ClipPlane](Docs/BP_ClipPlane.png "Blueprint Actor BP_ClipPlane")<br>*Fig. 2.3.: Blueprint Actor BP_ClipPlane*

### 2.4. Spot-Light

The plugin provides with a Blueprint SpotLight named `BP_StaticSpotLight` whose transform mobility is set to *static* (see figure 2.4.). Its `SpotLightComponent` *Light* parameters are simulating an operating theatre light:

* Intensity (Brightness): 1700 Lumen (see [UEDoc, Physical Lighting Units])
* Temperature: 5100 K (cp. [21])

Plugin Content:

* Blueprint Actor: `BP_StaticSpotLight`

![Blueprint SpotLight BP_StaticSpotLight](Docs/BP_StaticSpotLight.png "Blueprint SpotLight BP_StaticSpotLight")<br>*Fig. 2.4.: Blueprint SpotLight BP_StaticSpotLight*

### 2.5. Workflows

*Fig 2.1.: Sequence Diagramm, Import Workflow*
```mermaid
sequenceDiagram
  rect rgb(50, 200, 50)
  activate *.dcm
  activate Importer
  Importer->>*.dcm: Read
  activate T_SV_HU_Volume
  deactivate *.dcm
  Importer->>T_SV_HU_Volume: Save
  deactivate Importer
  deactivate T_SV_HU_Volume
  end
```

*Fig 2.2.: Sequence Diagramm, DVR Workflow - From Hounsfield Unit Volume with DICOM Window, TF and ROI*
```mermaid
sequenceDiagram
  rect rgb(0, 200, 255)
    T_SV_HU_Volume->>M_SV_W_CS: Param.
    DICOM_Window->>M_SV_W_CS: Param.
    M_SV_W_CS->>RT_SV_W_Volume: Compute
    T_TF_ColorAtlas->>M_DVR-Raycasting_W: Param.
    BP_ROI-SV->>M_DVR-Raycasting_W: Param.
    RT_SV_W_Volume->>M_DVR-Raycasting_W: Param.
    M_DVR-Raycasting_W->>BP_SV_HU: Render
    rect rgb(0, 100, 255)
      alt Variant - Save DICOM Windowed RT as Volume Texture, and use the same as Param.
        RT_SV_W_Volume-->>T_SV_W_Volume: Save
        T_TF_ColorAtlas->>M_DVR-Raycasting_W: Param.
        BP_ROI-SV->>M_DVR-Raycasting_W: Param.
        T_SV_W_Volume-->>M_DVR-Raycasting_W: Param.
        M_DVR-Raycasting_W->>BP_SV_W: Render
      end
    end
  end
```

*Fig 2.3.: Sequence Diagramm, DVR Workflow - Lighting from DICOM Windowed RT, TF and ROI*
```mermaid
sequenceDiagram
  rect rgb(255, 255, 200)
    RT_SV_W_Volume->>M_SV_L_CS: Param.
    M_SV_L_CS->>RT_SV_L_Volume: Compute
    T_TF_ColorAtlas->>M_DVR-Raycasting_WL: Param.
    BP_ROI-SV->>M_DVR-Raycasting_WL: Param.
    RT_SV_W_Volume->>M_DVR-Raycasting_WL: Param.
    RT_SV_L_Volume->>M_DVR-Raycasting_WL: Param.
    M_DVR-Raycasting_WL->>BP_SV_HU: Render
    rect rgb(255, 255, 100)
      alt Variant - Save Lighting RT as Volume Texture, and use the same as Param.
        RT_SV_L_Volume-->>T_SV_L_Volume: Save
        T_TF_ColorAtlas->>M_DVR-Raycasting_WL: Param.
        BP_ROI-SV->>M_DVR-Raycasting_WL: Param.
        T_SV_L_Volume-->>M_DVR-Raycasting_WL: Param.
        M_DVR-Raycasting_WL->>BP_SV_WL: Render
      end
    end
  end
```

*Fig 2.4.: Sequence Diagramm, MPR-3D/-2D Workflow - From DICOM Windowed RT Volume and LUT*
```mermaid
sequenceDiagram
  rect rgb(200, 200, 200)
    RT_SV_W_Volume->>M_MPR_CS: Param.
    rect rgb(150, 150, 150)
      alt Variant - Use Volume Texture
        T_SV_W_Volume-->>M_MPR_CS: Param.
      end
    end
    M_MPR_CS->>RT_MPR-Axial/-Coronal/-Sagittal: Compute
    T_LUT_Array->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
    RT_MPR-Axial/-Coronal/-Sagittal->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
    Plane-Axial/-Coronal/-Sagittal_Location->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
    Plane-ForwardVector/-UpVector->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
    MI_MPR-Axial/-Coronal/-Sagittal->>BP_MPR-3D/-2D: Render
  end
```

---

*Fig 2.10.: Sequence Diagramm, Workflow Overview*
```mermaid
sequenceDiagram
  opt Import Workflow
    rect rgb(50, 200, 50)
      activate *.dcm
      activate Importer
      Importer->>*.dcm: Read
      activate T_SV_HU_Volume
      deactivate *.dcm
      Importer->>T_SV_HU_Volume: Save
      deactivate Importer
      deactivate T_SV_HU_Volume
    end
  end

  opt DVR Workflow - From Hounsfield Unit Volume with DICOM Window, TF and ROI
    rect rgb(0, 200, 255)
      T_SV_HU_Volume->>M_SV_W_CS: Param.
      DICOM_Window->>M_SV_W_CS: Param.
      M_SV_W_CS->>RT_SV_W_Volume: Compute
      T_TF_ColorAtlas->>M_DVR-Raycasting_W: Param.
      BP_ROI-SV->>M_DVR-Raycasting_W: Param.
      RT_SV_W_Volume->>M_DVR-Raycasting_W: Param.
      M_DVR-Raycasting_W->>BP_SV_HU: Render
      rect rgb(0, 100, 255)
        alt Variant - Save DICOM Windowed RT as Volume Texture, and use the same as Param.
          RT_SV_W_Volume-->>T_SV_W_Volume: Save
          T_TF_ColorAtlas->>M_DVR-Raycasting_W: Param.
          BP_ROI-SV->>M_DVR-Raycasting_W: Param.
          T_SV_W_Volume-->>M_DVR-Raycasting_W: Param.
          M_DVR-Raycasting_W->>BP_SV_W: Render
        end
      end
    end
  end

  opt DVR Workflow - Lighting from DICOM Windowed RT, TF and ROI
    rect rgb(255, 255, 200)
      RT_SV_W_Volume->>M_SV_L_CS: Param.
      M_SV_L_CS->>RT_SV_L_Volume: Compute
      T_TF_ColorAtlas->>M_DVR-Raycasting_WL: Param.
      BP_ROI-SV->>M_DVR-Raycasting_WL: Param.
      RT_SV_W_Volume->>M_DVR-Raycasting_WL: Param.
      RT_SV_L_Volume->>M_DVR-Raycasting_WL: Param.
      M_DVR-Raycasting_WL->>BP_SV_HU: Render
      rect rgb(255, 255, 100)
        alt Variant - Save Lighting RT as Volume Texture, and use the same as Param.
          RT_SV_L_Volume-->>T_SV_L_Volume: Save
          T_TF_ColorAtlas->>M_DVR-Raycasting_WL: Param.
          BP_ROI-SV->>M_DVR-Raycasting_WL: Param.
          T_SV_L_Volume-->>M_DVR-Raycasting_WL: Param.
          M_DVR-Raycasting_WL->>BP_SV_WL: Render
        end
      end
    end
  end

  opt MPR-3D/-2D Workflow - From DICOM Windowed RT Volume and LUT
    rect rgb(200, 200, 200)
      RT_SV_W_Volume->>M_MPR_CS: Param.
      rect rgb(150, 150, 150)
        alt Variant - Use Volume Texture
          T_SV_W_Volume-->>M_MPR_CS: Param.
        end
      end
      M_MPR_CS->>RT_MPR-Axial/-Coronal/-Sagittal: Compute
      T_LUT_Array->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
      RT_MPR-Axial/-Coronal/-Sagittal->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
      Plane-Axial/-Coronal/-Sagittal_Location->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
      MPR-3D_Plane-ForwardVector/-UpVector->>MI_MPR-Axial/-Coronal/-Sagittal: Param.
      MI_MPR-Axial/-Coronal/-Sagittal->>BP_MPR-3D/-2D: Render
    end
  end
```

## 3. Blueprint SV and Inheriting Actors

The scalar volume datasets are handled in actors based on an abstract Blueprint named `BPA_SV`, which holds a `StaticMeshComponent` of type `Cube` named `SVMesh` and another `StaticMeshComponent` of type `Cube` named `BoundingBox` (see Class Diagram in figure 3.1.).

Plugin Content:

* Blueprint Actors: `BPA_SV`, `BP_SV_HU`, `BP_SV_W`, `BP_SV_WL`

*Fig 3.1.: Blueprint Inheritance Diagramm*
```mermaid
classDiagram
  class BPA_SV{
    <<Abstract>>
    +StaticMeshComponent Cube : SVMesh
    +StaticMeshComponent Cube : BoundingBox
    +F_SV : VolumeRendering
    +E_VolumeRenderingMethod : Method
    +MaterialInstanceDynamic : MI_SV_Dynamic
    +RescaleSVMesh(F_PixelSpacing PixelSpacing)
  }
  class BP_SV_HU{
    +F_SV_HU : Dataset
    -TextureRenderTargetVolume : WindowRT
    -TextureRenderTargetVolume : LightmapRT
    #ConstructionScript()
    +ComputeWindowRT()
    +ComputeLightmapRT()
    +SaveWindowRT()
    +SaveLightmapRT()
    +EventDispatcher : OnChangedWindowRT()
    +EventDispatcher : OnChangedLightmapRT()
    +Update_WindowVolume()
  }
  class BPI_SV_W{
    <<Interface>>
    +Update_WindowVolume()
    +Update_RegionOfInterest()
    +Update_ClipPlane()
    +Update_DistancePower()
    +Update_Steps()
    +Update_TransferFunction()
    +Update_AlphaMax()
  }
  class BPI_SV_L{
    <<Interface>>
    +Update_LightmapVolume()
    +Update_Ambient()
    +Update_Diffuse()
    +Update_Specular()
    +Update_SpecularPower()
  }
  class BP_SV_W{
    +F_SV_W : Dataset
    #ConstructionScript()
    +Update_WindowVolume()
  }
  class BP_SV_WL{
    +F_SV_L : Shading
    #ConstructionScript()
  }

  BPA_SV <|-- BP_SV_HU : Inherits
  BPI_SV_W <|.. BPA_SV : Implements
  BPI_SV_L <|.. BP_SV_HU : Implements

  BPI_SV_L <|.. BP_SV_WL : Implements

  BPA_SV <|-- BP_SV_W : Inherits
  BP_SV_W <|-- BP_SV_WL : Inherits  

```
  
*Fig 3.2.: Struct Composition Diagramm*
```mermaid
classDiagram
  class F_SV{
    +BP_ROI : RegionOfInterest
    +BP_ClipPlane : ClipPlane
    +Float : DistancePower
    +Integer : ResamplingSteps
    +CurveLinearColor : TransferFunction
    +Float : AlphaMax
  }
  class F_SV_HU{
    +VolumeTexture : HounsfieldVolume
    +F_PixelSpacing : PixelSpacing
    +F_DicomWindow : DicomWindow
    +Boolean : WindowMask
    +F_Phong : PhongShading
    +F_Lightmap : Lightmap
  }
  class F_SV_W{
    +VolumeTexture : WindowVolume
    +F_PixelSpacing : PixelSpacing
    +F_DicomWindow : DicomWindow
  }
  class F_SV_L{
    +VolumeTexture : LightmapVolume
    +F_Phong : PhongShading
  }
  class F_PixelSpacing{
    +Integer : Columns
    +Float : ColumnSpacing
    +Integer : Rows
    +Float : RowSpacing
    +Integer : Slices
    +Float : SliceThickness
  }
  class F_DicomWindow{
    +F_WindowLocation : Location
    +F_WindowBorder : Border
  }
  class F_WindowLocation{
    +Integer : Center
    +Integer : Width
  }
  class F_WindowBorder{
    +Integer : Left
    +Integer : Right
  }
  class F_Phong{
    +Float : Ambient
    +Float : Diffuse
    +Float : Specular
    +Integer : SpecularPower
  }
  class F_Lightmap{
    +Boolean : EnableLighting
    +BP_StaticSpotLight Array : SpotLights
    +Boolean : HalfResolution
  }

  F_SV_HU --* F_PixelSpacing
  F_SV_HU --* F_DicomWindow
  F_SV_HU --* F_Lightmap
  F_SV_HU --* F_Phong

  F_SV_W --* F_PixelSpacing
  F_SV_W --* F_DicomWindow

  F_SV_L --* F_Phong

  F_DicomWindow --* F_WindowLocation
  F_DicomWindow --* F_WindowBorder
```

### 3.1. Actor BP SV H

#### 3.1.2. Dataset

An image-stack based volume&mdash;commonly known as scalar volume&mdash;is kept as Volume Texture asset in Unreal Engine.

#### 3.1.3. DICOM Window

CT image data is expected to come in Hounsfield Units HU in a range of [-1000, 3095] (cp. section Import) representing 4096 gray levels for different materials where air is defined as -1000 HU and water as 0 HU. Consumer computer screens only can visualize 256 gray levels, represented by a value range of [0, 255]. Therefore the 4096 Hounsfield Units are mapped to the 256 screen gray scale levels. This is done by linear interpolation (Lerp).

If the whole range of 4096 Hounsfield data is mapped to 256 gray levels, the contrast becomes quite bad. Therefore, the so called DICOM Window was introduced to downsize the range of Hounsfield data to map.

To allow to render the lerped values only, a mask may be applied to the volume's Hounsfiled data. Values lesser than the window left border are mapped to 0, greater than the window right border are mapped to 255.

#### 3.1.4. Volume Rendering

Direct Volume Rendering DVR with Materials from Raymarching Shaders with static lighting.

##### 3.1.4.1. Region Of Interest

* MeshCube: `BP_ROI` instance as Reference Object, ideally subordinated in Outline Hierarchy (Scene Graph)
* used for geometry subtraction in the shader

##### 3.1.4.2. Clip Plane

* MeshPlane: `BP_ClipPlane` instance as object as Reference Object
* used for geometry subtraction in the shader

##### 3.1.4.3. Distance Power

* Default Value: `1.0`
* Range: [`0.1`, `2.0`]
* Resampling Distance Power:
  * The shader algorithm calculates the current distance of the image slices with respect to the angle of entry of the resampling ray. With a value of `1.0` (default) the calculated resampling distance is used.
  * With values smaller than `1.0` the resampling distance lowers, a so-called oversampling occurs, which may increase visualisation quality.
  * With values larger than `1.0` the resampling distance grows, a so-called undersampling occurs, which may accelerate rendering.

This parameter may be seen as an optimisation method, cp. [Luecke 2005], *Fragmented Line Ray-Casting*:
> *To lower the number of operations necessary for computing a single frame, [...] the distance between two successive resampling locations, i.e the sampling distance, could be increased, thereby decreasing the number of actual locations used for volume reconstruction.*
> *However, it is worth mentioning, that incorporating any of these optimization approaches usually tends to result in generated images of less quality compared to an unoptimized ray-casting volume renderer.*

##### 3.1.4.4. Resampling Steps

* Default Value: `256`
* Range: [`1`, `1024`]
* Maximum Number of Resampling Steps:
  * A large number means more steps. The resampling ray may advance deeper into the cube. The hereby resulting rendering may increase visualisation quality by the cost of more computing time.
  * A small number may decrease rendering quality but is faster.

##### 3.1.4.5. Transfer Function

The transfer functions are based on color gradients from `Curve Linear Color` assets, bundled in a Texture 2D `Curve Atlas` asset as Look-Up Table LUT:

* Curve Linear Color *TF* assets named `Curve_TF-[*]_Color`
* Curve Atlas *TF-LUT* asset named `T_Curve_TF-LUT_ColorAtlas`

The gradients represent values as found in 3D-Slicer&trade; Module "Volume Rendering" (cp. [Finet et al.]).

##### 3.1.4.6. Alpha Max

Maximum Opacity Threshold for Early Ray Termination

* Maximum Value from Iteratively added up Alpha Channel
* Default Value: `0.8`
* Range: [`0.0`, `1.0`]

<div style='page-break-after: always'></div>

#### 3.1.5. Volume Shading

https://help.maxon.net/r3d/katana/en-us/Content/html/IES+Light.html#CommonRedshiftLightParameters-DiffuseScale

##### 3.1.5.1. Phong

* Ambient: Ambient Reflection Value in [`0.0`, `1.0`], Default `0.1`
* Diffuse: Diffuse Reflection Value in [`0.0`, `1.0`], Default `0.9`
* Specular: Specular Reflection Value in [`0.0`, `1.0`], Default `0.2`
* Specular Power: Specular Reflection Power Value  in [`1`, `50`], Default `10`

##### 3.1.5.2. Lighting

* Spot Lights: Array of `BP_StaticSpotLight` Object Reference
* Half Resolution: Default `true` (checked)
* Lightmap Volume: Texture Render Target Volume Object Reference, Default `RT_SV_L_Volume`

<div style='page-break-after: always'></div>

### 3.2. Actor BP SV W

<div style='page-break-after: always'></div>

### 3.3. Actor BP SV W L

<div style='page-break-after: always'></div>

## 4. Import

CT image data is expected to come in Hounsfield Units HU as values in a range of [-1000, 3095] which are 4096 gray levels for different materials. These 4096 gray levels can be optimally represented with a twelve-digit binary number (12 bit, $2^{12} = 4096$).

Naming Convention: Underlines in file names (`_`) are replaced by minus in asset names (`-`)

### 4.1. Import DICOM

DICOM&reg; *.dcm

The results are stored in a Render Target Volume named `RT_Scalar_Volume`, R-channel.

### 4.2. Import MetaImage

MetaImage&trade; *.mhd

<div style='page-break-after: always'></div>

### 4.3. Data Background

#### 4.3.1. Memory

Scalar volume size $V_1$ (cp. [DICOM, FAQ]):

* A Stack of 256 images of size 256 x 256 pixel per image = 256<sup>3</sup> pixel or voxel resp.
* 4 channels RGBA
* With 8 bit per channel ($2^{8} = 256$, range from 0 to 255)

$ V_1 = 256^3 \times 4 \times 8\ {}bit = 536’870’912\ {}bit = 0.537\ {}Gigabit = 67\ {}MB $

If the images are double the size (stack of 512 images with 512 x 512 pixel per image), the size $V_2$ increases to 0.5 GB:

$ V_2 = 512^3 \times 4 \times 8\ {}bit = 4’294’967’296\ {}bit = 4.295\ {}Gigabit = 537\ {}MB $

If the images are double the size (stack of 1024 images with 1024 x 1024 pixel per image), the size $V_3$ increases to 4 GB:

$ V_3 = 1024^3 \times 4 \times 8\ {}bit = 34’359’738’368\ {}bit = 34.359\ {}Gigabit = 4295\ {}MB $

#### 4.3.2. Processing

With processing, e.g., $30 \text{ fps}$:

$
Processed\ {}Data_1 = \frac{0.537\ {}Gigabit}{frame} \times \frac{30\ {}frames}{s} = 16.1\ {} Gigabit\ {}per\ {}second
$

$
Processed\ {}Data_2 = \frac{4.295\ {}Gigabit}{frame} \times \frac{30\ {}frames}{s} = 128.8\ {} Gigabit\ {}per\ {}second
$

$
Processed\ {}Data_3 = \frac{34.359\ {}Gigabit}{frame} \times \frac{30\ {}frames}{s} = 1030.8\ {} Gigabit\ {}per\ {}second
$

<!-- 
https://www.quora.com/How-can-a-processor-handle-10-Gigabit-per-second-or-more-data-rate
-->

<div style='page-break-after: always'></div>

## 6. Outlook

Not yet implmeneted features:

* MPR-2D
  * Orientation Guide
  * Region of Interest visualization
  * Plane Interaction
* Rendering Type:
  * Indirect Volume Rendering
  * Vector Volume Rendering &ndash; where the voxels store multiple scalar values, e.g., LPS or RAS coordinates as components of a displacement field (cp. [Piper et al., Overview]).
  * Tensor Volume Rendering &ndash; where the voxels store a tensor, e.g., used for MRI diffusion tensor imaging DTI (cp. [Piper et al., Overview]).
  <!--* Labelmap Volume Rendering &ndash; where the voxels store a discrete value, such as an index or a label; e.g., used for segmentation.-->

<div style='page-break-after: always'></div>

## Appendix

### Abbreviations and Acronyms

* A &mdash; Anterior
* ARS &mdash; Anterior&ndash;Right&ndash;Superior
* AXE &mdash; Axial
* BB &mdash; Bounding Box
* COR &mdash; Coronal
* CS &mdash; Compute Shader
* CT &mdash; Computed Tomography (X-ray)
* DICOM &mdash; Digital Imaging and Communications in Medicine
* DVR &mdash; Direct Volume Rendering
* fps &mdash; Frames per Second
* FPV &mdash; First Person View
* HU &mdash; Hounsfield Units
* I &mdash; Inferior
* L &mdash; Left
* LAS &mdash; Left&ndash;Anterior&ndash;Superior
* LhS &mdash; Left-handed System
* LPS &mdash; Left&ndash;Posterior&ndash;Superior
* LUT &mdash; Look-Up Table
* MIP &mdash; Maximum Intensity Projection
* MPR &mdash; Multiplanar Reconstruction
* MR &mdash; Magnetic Resonance
* OG &mdash; Orientation Guide
* P &mdash; Posterior
* R &mdash; Right
* RAS &mdash; Right&ndash;Anterior&ndash;Superior
* RhS &mdash; Right-handed System
* ROI &mdash; Region of Interest
* S &mdash; Superior
* SAG &mdash; Sagittal
* SV &mdash; Scalar Volume
* TF &mdash; Transfer Function
* US &mdash; Ultrasound Imaging (sonography)

<!--
* AAA &mdash; Abdominal Aortic Aneurysm
* CRI &mdash; Colour Rendering Index
* CTA &mdash; Computed Tomography Angiography
* IVR &mdash; Indirect Volume Rendering
* MRI &mdash; Magnetic Resonance Imaging
* MRT &mdash; Magnetic Resonance Tomography
* PET &mdash; Positron Emission Tomography
* RCS &mdash; Reference Coordinate System
* VOI &mdash; Volume of Interest
-->

<div style='page-break-after: always'></div>

### Glossary

#### Terms of Location and Coordinate Systems

Anatomical planes and terms of location on a person standing upright (cp. [mbbs]):

* **Sagittal Plane**: The median plane is a longitudinal plane, which separates the body into its **Right (R)** and **Left (L)** halves. A sagittal plane is any plane perpendicular to the median plane.
* **Coronal Plane**: Frontal plane, separates in **Posterior (P)** towards back and **Anterior (A)** towards front.
* **Axial Plane**: Horizontal (transverse) plane, separates in **Inferior (I)** towards feet (bottom) and **Superior (S)** towards head (top).

##### DICOM

DICOM images are using a **Left&ndash;Posterior&ndash;Superior LPS** system (cp. [Sharma 2022] and [Adaloglouon 2020], *Anatomical coordinate system*):
> *"[Left&ndash;Posterior&ndash;Superior] LPS is used by DICOM images and by the ITK toolkit, while 3D Slicer and other medical software use [Right&ndash;Anterior&ndash;Superior] RAS"*

* **L**: X increases from R to L
* **P**: Y increases from A to P
* **S**: Z increases from I to S

DICOM images are using a **Right-handed System RhS** of matrix or index coordinates as rows of columns of pixel values in a stack of slices (cp. [Adaloglouon 2020], *Medical Image coordinate system (Voxel space)*):

* i: Image width in columns, increases to the right
* j: Image height in rows, increases downwards
* k: Image stack depth in slices, increases backwards

##### Unreal Engine

Unreal Engine is using a **Left-handed System LhS** based First Person View FPV (cp. [Mower, Coordinate System]):

* X: Increases from **Back** to **Front**, color code red
* Y: Increases from **Left** to **Right**, color code green
* Z: Increases upwards from **Bottom** to **Top**, color code blue

###### Plugin "Volume Creator"

The anatomical coordinate system in plugin "Volume Creator"&mdash;with UE's use of a LhS&mdash;results in an **Anterior&ndash;Right&ndash;Superior ARS** anatomical coordinate system (cp. figure G.1.):

* **A**: X increases from Back to Front, color code red; anatomical from **Posterior (P)** to **Anterior (A)**
* **R**: Y increases from Left to Right, color code green; anatomical from  **Left (L)** to **Right (R)**
* **S**: Z increases upwards from Bottom to Top, color code blue; anatomical from **Inferior (I)** to **Superior (S)**

![DVR Orientation Guide Actor with Left Handed UE-Location-Gizmo Arrows](Docs/OrientationGuide.png "DVR Orientation Guide Actor with Left Handed UE-Location-Gizmo Arrows")<br>*Fig. G.1.: DVR Orientation Guide Actor with Left Handed UE-Location-Gizmo Arrows*

Anatomical Planes and Terms of Location in this plugin (cp. figure G.2.):

* **Coronal (COR)**: Frontal **YZ-Plane** (green/blue arrows) with **Up-Vector X+** (red arrow) from **Posterior (P)** to **Anterior (A)**
* **Sagittal (SAG)**: Longitudinal **XZ-Plane** (red/blue arrows) with **Up-Vector Y+** (green arrow) from **Left (L)** to **Right (R)**
* **Axial (AXE)**: Horizontal **XY-Plane** (red/green arrows) with **Up-Vector Z+** (blue arrow) from **Inferior (I)** to **Superior (S)**

![ROI-Handles Actor with Left Handed UE-Location-Gizmo Arrows](Docs/ROIHandles.png "ROI-Handles Actor with Left Handed UE-Location-Gizmo Arrows")<br>*Fig. G.2.: ROI-Handles Actor with Left Handed UE-Location-Gizmo Arrows*

<div style='page-break-after: always'></div>

#### Asset Naming Convention

The plugins assets naming convention is based on a scheme from [UEDoc, Recommended Asset Naming Conventions] as well as from [Allar 2022]:
> *`[AssetTypePrefix]_[AssetName]_[DescriptorSuffix]_[OptionalVariantLetterOrNumber]`*
>
>* *`AssetTypePrefix` identifies the type of Asset [...].*
>* *`AssetName` is the Asset's name.*
>* *`DescriptorSuffix` provides additional context for the Asset, to help identify how it is used. For example, whether a texture is a normal map or an opacity map.*
>* *`OptionalVariantLetterOrNumber` is optionally used to differentiate between multiple versions or variations of an asset.*

* `[AssetTypePrefix]` (UEDoc and Allar):
  * Blueprint: `BP`
  * Blueprint Interface: `BPI`
  * Curve: `Curve`
  * Enum(eration): `E`
  * Material: `M`
  * Material Instance: `MI`
  * Material Instance Dynamic: `MID`
  * Struct(ure): `F`
  * Texture: `T`
  * Texture Render Target: `RT`
* `[AssetName]` (Domain Specific):
  * Scalar Volume: `SV`
  * Acquisition Type:
    * Computer Tomography: `CT`
    * Magnetic Resonance: `MR`
    * Ultrasound: `US`
  * Data Type:
    * Histogram: `HIS`
    * Hounsfield Units: `HU`
    * DICOM Window: `W`
    * Lighting: `L`
  * Rendering Type:
    * Multiplanar Rendering: `MPR`
      * Plane: `COR`, `SAG`, `AXE`
      * Location: `P`, `A`, `L`, `R`, `I`, `S`
      * Look-Up Table: `LUT`
    * Direct Volume Rendering: `DVR`
      * Bounding Box: `BB`
      * Orientation Guide: `OG`
      * Region of Interest: `ROI`
      * Transfer Function: `TF`
* `[DescriptorSuffix]` (UEDoc and Allar):
  * Texture Array: `Array`
  * Curve Linear Color: `Color`
  * Color Atlas: `ColorAtlas`
  * Compute Shader: `CS`
  * Main Material: `Main`
  * Volume Texture: `Volume`
  * Texture Drawn from 'Material to Texture Render Target': `Tex`

Aditional Conventions:

* In the `[AssetName]`, dashes "`-`" are used, no underlines "`_`".
* In the `[AssetName]`, single letter suffixes are combined without additional dashes "`-`".

<div style='page-break-after: always'></div>

### A. References

#### A.1. Medical Imaging

* Anatomical Terms:
  * [mbbs] mbbsbooks: **Anatomical Terms**. In: mbbsbooks Medical - Category Anatomy. Feb 14, 2023. Online: [https://mbbsbooks.com/anatomical-terms/](https://mbbsbooks.com/anatomical-terms/)
* DICOM:
  * [DICOM] **The DICOM Standard**. Online: [https://www.dicomstandard.org/current](https://www.dicomstandard.org/current)
  * [DICOM, FAQ] **DICOM Standard FAQ**. Online: [https://www.dicomstandard.org/faq](https://www.dicomstandard.org/faq)
  * [DICOM-Browser] Innolitics: **DICOM Standard Browser**. Online: [https://dicom.innolitics.com/ciods/ct-image](https://dicom.innolitics.com/ciods/ct-image)
  * [Sharma 2021] Shivam Sharma: **Introduction to DICOM for Computer Vision Engineers**. In: *RedBrick AI*. Dec 15, 2021. Online: [https://medium.com/redbrick-ai/introduction-to-dicom-for-computer-vision-engineers-78f346bbc1fd](https://medium.com/redbrick-ai/introduction-to-dicom-for-computer-vision-engineers-78f346bbc1fd)
  * [Sharma 2022] Shivam Sharma: **DICOM Coordinate Systems &ndash; 3D DICOM for Computer Vision Engineers**. In: *RedBrick AI*. Dec 22, 2022. Online: [https://medium.com/redbrick-ai/dicom-coordinate-systems-3d-dicom-for-computer-vision-engineers-pt-1-61341d87485f](https://medium.com/redbrick-ai/dicom-coordinate-systems-3d-dicom-for-computer-vision-engineers-pt-1-61341d87485f)
  * [Adaloglouon 2020] Nikolas Adaloglouon: **Understanding Coordinate Systems and DICOM for Deep Learning Medical Image Analysis**. In: *The AI Summer*. July 16, 2020. Online: [https://theaisummer.com/medical-image-coordinates/](https://theaisummer.com/medical-image-coordinates/)
  * [Zaharia 2013] Roni Zaharia: **Chapter 14 - Image Orientation: Getting Oriented using the Image Plane Module**. In: *DICOM Tutorial, DICOM is Easy &ndash; Software Programming for Medical Applications*. June 6, 2013. Online: [http://dicomiseasy.blogspot.com/2013/06/getting-oriented-using-image-plane.html](http://dicomiseasy.blogspot.com/2013/06/getting-oriented-using-image-plane.html)
* Volume Rendering:
  * [Engel et al. 06] Klaus Engel, Markus Hadwiger, Joe Kniss, Christof Rezk Salama, Daniel Weiskopf (2006): **Real-Time Volume Graphics**. doi: [10.1145/1103900.1103929](http://dx.doi.org/10.1145/1103900.1103929). Online: [http://www.real-time-volume-graphics.org/](http://www.real-time-volume-graphics.org/)
  * [Hadwiger et al. 18] Markus Hadwiger, Ali K. Al-Awami, Johanna Beyer, Marcos Agos, Hanspeter Pfister (2018): **SparseLeap: Efficient Empty Space Skipping for Large-Scale Volume Rendering**. In: *IEEE Transactions on Visualization and Computer Graphics*. Online: [https://vcg.seas.harvard.edu/publications/sparseleap-efficient-empty-space-skipping-for-large-scale-volume-rendering](https://vcg.seas.harvard.edu/publications/sparseleap-efficient-empty-space-skipping-for-large-scale-volume-rendering)
  * [Luecke 2005] Peter Lücke: **Volume Rendering Techniques for Medical Imaging**. Diplomarbeit. Technische Universität München, Fakultät für Informatik. April 15, 2005. In collaboration with Siemens Corporate Research Inc., Princeton, USA. Online: [https://campar.in.tum.de/twiki/pub/Students/DaLuecke/Diplomarbeit.pdf](https://campar.in.tum.de/twiki/pub/Students/DaLuecke/Diplomarbeit.pdf)
  * [Piper et al.] Steve Piper (Isomics), Julien Finet (Kitware), Alex Yarmarkovich (Isomics), Nicole Aucoin (SPL, BWH): **3D Slicer Module "Volumes"**. License: slicer4. The work is part of the National Alliance for Medical Image Computing (NAMIC), funded by the National Institutes of Health through the NIH Roadmap for Medical Research, Grant U54 EB005149. Online Documentation: [https://slicer.readthedocs.io/en/latest/user_guide/modules/volumes.html](https://slicer.readthedocs.io/en/latest/user_guide/modules/volumes.html)
  * [Finet et al.] Julien Finet (Kitware), Alex Yarmarkovich (Isomics), Yanling Liu (SAIC-Frederick, NCI-Frederick), Andreas Freudling (SPL, BWH), Ron Kikinis (SPL, BWH): **3D Slicer Module "Volume Rendering"**. License: slicer4. The work is part of the National Alliance for Medical Image Computing (NAMIC), funded by the National Institutes of Health through the NIH Roadmap for Medical Research, Grant U54 EB005149. Online Documentation: [https://slicer.readthedocs.io/en/latest/developer_guide/modules/volumerendering.html](https://slicer.readthedocs.io/en/latest/developer_guide/modules/volumerendering.html); Transfer Function Presets on GitHub: [https://github.com/Slicer/Slicer/blob/main/Modules/Loadable/VolumeRendering/Resources/presets.xml](https://github.com/Slicer/Slicer/blob/main/Modules/Loadable/VolumeRendering/Resources/presets.xml)
* Lighting:
  * [21] **Why Colour Matters in Surgical Lighting**. In: Website of Vivo Surgical. Jul 27, 2021. Online: [https://www.vivo-surgical.com/post/why-colour-matters-the-importance-of-colour-temperature](https://www.vivo-surgical.com/post/why-colour-matters-the-importance-of-colour-temperature)
  <!--* [22] **The Different Colors Of Operating Theatre Lights**. In: Website "Forum Theatre". September 15, 2022. Online: [https://forum-theatre.com/the-different-colors-of-operating-theatre-lights/](https://forum-theatre.com/the-different-colors-of-operating-theatre-lights/)-->

#### A.2. Unreal Engine

* [UEDoc] Epic Games: **Unreal Engine Documentation**. URL: [https://docs.unrealengine.com](https://docs.unrealengine.com)
* Coordinate System:
  * [Mower, Scale] Nick Mower: **Scale and Measurement Inside Unreal Engine 4**. In: TechArt-Hub. Online: [https://www.techarthub.com/scale-and-measurement-inside-unreal-engine-4/](https://www.techarthub.com/scale-and-measurement-inside-unreal-engine-4/)
  * [Mower, Coordinate System] Nick Mower: **A Practical Guide to Unreal Engine 4’s Coordinate System**. In: TechArt-Hub. Online: [https://www.techarthub.com/a-practical-guide-to-unreal-engine-4s-coordinate-system/](https://www.techarthub.com/a-practical-guide-to-unreal-engine-4s-coordinate-system/)
* Naming Convention:
  * [UEDoc, Recommended Asset Naming Conventions] Epic Games: **Recommended Asset Naming Conventions**. In: Unreal Engine Documentation. URL: [https://docs.unrealengine.com/5.1/en-US/recommended-asset-naming-conventions-in-unreal-engine-projects/](https://docs.unrealengine.com/5.1/en-US/recommended-asset-naming-conventions-in-unreal-engine-projects/)
  * [Allar 2022] Michael Allar: **Gamemakin UE Style Guide**. Mar 7, 2022. URL: [https://github.com/Allar/ue5-style-guide](https://github.com/Allar/ue5-style-guide)
* Textures:
  * [UEDoc, Guidelines for Optimizing Rendering for Real-Time] Epic Games: **Guidelines for Optimizing Rendering for Real-Time**. In: Unreal Engine Documentation. URL: [https://docs.unrealengine.com/5.1/en-US/guidelines-for-optimizing-rendering-for-real-time-in-unreal-engine/](https://docs.unrealengine.com/5.1/en-US/guidelines-for-optimizing-rendering-for-real-time-in-unreal-engine/)
  * [Mower, Compression] Nick Mower: **Your Guide to Texture Compression in Unreal Engine**. In: TechArt-Hub. Online: [https://www.techarthub.com/your-guide-to-texture-compression-in-unreal-engine/](https://www.techarthub.com/your-guide-to-texture-compression-in-unreal-engine/)
  * [Ivanov 2021] Michael Ivanov: **Unreal Engine and Custom Data Textures**. Jun 19, 2021 URL: [https://sasmaster.medium.com/unreal-engine-and-custom-data-textures-40857f8b6b81](https://sasmaster.medium.com/unreal-engine-and-custom-data-textures-40857f8b6b81)
* Lighting:
  * [UEDoc, Physical Lighting Units] **Physical Lighting Units**. In: Unreal Engine Documentation. URL: [https://docs.unrealengine.com/4.27/en-US/BuildingWorlds/LightingAndShadows/PhysicalLightUnits/](https://docs.unrealengine.com/4.27/en-US/BuildingWorlds/LightingAndShadows/PhysicalLightUnits/)

### B. Readings

* [Ikits et al. 2007] Milan Ikits, Joe Kniss, Aaron Lefohn, Charles Hansen: **Volume Rendering Techniques**. In: *GPU Gems: Programming Techniques, Tips, and Tricks for Real-Time Graphics &ndash; Part VI: Beyond Triangles, Chapter 39*. 5th Printing September 2007, Pearson Education, Inc. Online: [https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-39-volume-rendering-techniques](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-39-volume-rendering-techniques)
<!-- * Fedorov A., Beichel R., Kalpathy-Cramer J., Finet J., Fillion-Robin J-C., Pujol S., Bauer C., Jennings D., Fennessy F.M., Sonka M., Buatti J., Aylward S.R., Miller J.V., Pieper S., Kikinis R: **3D Slicer as an Image Computing Platform for the Quantitative Imaging Network**. Online: [https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3466397/pdf/nihms383480.pdf](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3466397/pdf/nihms383480.pdf). Magnetic Resonance Imaging. 2012 Nov;30(9):1323-41. PMID: 22770690. PMCID: PMC3466397. -->

### C. Acknowledgements

* **Software:** Bruggmann, Roland (2023): **Volume Creator**, Version v1.0.0, UE 4.26&ndash;5.1. Unreal&reg; Marketplace. URL: [https://www.unrealengine.com/marketplace/en-US/product/volume-creator](https://www.unrealengine.com/marketplace/en-US/product/volume-creator). Copyright 2023 Roland Bruggmann aka brugr9. All Rights Reserved.
<!--
* **Data:** van Ginneken, Bram, & Jacobs, Colin. (2019): **LUNA16 Part 1/2 subset0**. Zenodo. [https://doi.org/10.5281/zenodo.3723295](https://doi.org/10.5281/zenodo.3723295), licensed under Creative Commons Attribution 4.0 International ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/))
-->

<div style='page-break-after: always'></div>

### D. Attribution

* The word mark *Unreal* and its logo are Epic Games, Inc. trademarks or registered trademarks in the US and elsewhere (cp. Branding Guidelines and Trademark Usage, URL: [https://www.unrealengine.com/en-US/branding](https://www.unrealengine.com/en-US/branding))
* The word mark *DICOM&mdash;Digital Imaging and Communication in Medicine* and its logo are trademarks or registered trademarks of the National Electrical Manufacturers Association (NEMA), managed by the Medical Imaging Technology Association (MITA), a division of NEMA
* The word mark *MetaImage* is a trademark or registered trademark of Kitware, Inc.
* The word mark *ITK&mdash;Insight Toolkit* is a trademark or registered trademark of Kitware, Inc.
* The word mark *3D Slicer* and the logo are trademarks of Brigham and Women’s Hospital (BWH), used with permission.

### E. Disclaimer

This documentation has **not been reviewed or approved** by the Food and Drug Administration FDA or by any other agency. It is the users responsibility to ensure compliance with applicable rules and regulations&mdash;be it in the US or elsewhere.

Read also:

* *"Documentation Disclaimer"* (file DISCLAIMER.md), Online: [https://github.com/brugr9/UEPluginVolumeCreator/blob/main/DISCLAIMER.md](https://github.com/brugr9/UEPluginVolumeCreator/blob/main/DISCLAIMER.md)
* *"Software Disclaimer"* from Plugin folder Docs/DISCLAIMER.pdf

### F. Citation

**Software**: To acknowledge *"Unreal&reg; Engine Plugin: Volume Creator"* software, please cite

> Bruggmann, Roland (2023). *Unreal&reg; Engine Plugin: Volume Creator*, Version [v#.#.#], UE [4.## or 5.#]. Unreal&reg; Marketplace. URL: [https://www.unrealengine.com/marketplace/en-US/product/volume-creator](https://www.unrealengine.com/marketplace/en-US/product/volume-creator). Copyright 2023 Roland Bruggmann aka brugr9. All Rights Reserved.

**Documentation**: To acknowledge this documentation&mdash;be it, e.g., the Readme or the Changelog&mdash;please cite

> Bruggmann, Roland (2023). *Volume Creator: An Unreal&reg; Engine Plugin for Medical Data Rendering &mdash; Documentation*, \[Readme, Changelog\]. GitHub; accessed [Year Month Day]. URL: [https://github.com/brugr9/UEPluginVolumeCreator](https://github.com/brugr9/UEPluginVolumeCreator). Licensed under [Creative Commons Attribution-ShareAlike 4.0 International](http://creativecommons.org/licenses/by-sa/4.0/)

---
<!-- Footer -->

[![Creative Commons Attribution-ShareAlike 4.0 International License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/)

*"Volume Creator: An Unreal&reg; Engine Plugin for Medical Data Rendering &mdash; Documentation"*. URL: [https://github.com/brugr9/UEPluginVolumeCreator](https://github.com/brugr9/UEPluginVolumeCreator). &copy; 2023 by [Roland Bruggmann](https://about.me/rbruggmann), licensed under [Creative Commons Attribution-ShareAlike 4.0 International](http://creativecommons.org/licenses/by-sa/4.0/)

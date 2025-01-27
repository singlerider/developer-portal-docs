---
id: remote-render-unreal-engine-5
title: Remote Rendering with Unreal Engine 5
sidebar_label: Unreal Engine 5
sidebar_position: 2
description: Remote Rendering with Unreal Engine 5.
date: 05/01/2023
tags: [Rendering, Servers, Unreal, UE, Remote Render]
keywords: [Rendering, Servers, Unreal, UE, Remote Render]
---

:::caution Performance and Gotchas
Performance for Remote Rendering is highly-dependent on the capabilities of the host machine that is running Unreal Engine. Noticeable lag may be a result of insufficient processing or GPU power.

In the future, an official plugin will provide more accurate mappings for Controller input.
:::

## Prerequisite

[Remote Rendering Setup Instructions with the Remote Rendering Service](/docs/guides/remote-render/remote-render).

## Setup

1. Create a "New Project" with the **Virtual Reality Template**

![Project Browser](/img/unreal-5/project-browser.png)

:::info
It is recommended to start the Remote Rendering service ahead of time. But, if it’s not running yet, the Remote Rendering service will be started automatically.
:::

1. Enable **OpenXR**

:::info
If you’re using the Virtual Reality template in Step 1, this step will not be necessary. When not using VR template, make sure to go through those setting before proceeding.
:::

- Make sure the **OpenXR** plugin is installed in **Edit** -> **Plugins**

![OpenXR Plugins Menu](/img/unreal-5/plugins-menu-openxr.png)

- Make sure `Start in VR` is enabled in **Edit** -> **Project Settings**

![Project Settings VR](/img/unreal-5/preoject-settings-vr.png)

2. Set the **RHI** to `Vulkan`

Under **Edit** -> **Project Settings**:

- Under **Platforms** in the left sidebar, select **Windows**
- Under **Targeted RHIs**, select `Vulkan` as the `Default RHI`

![Project Settings Platform Windows RHI](/img/unreal-5/project-settings-platform-windows.png)

3. Click <kbd>Restart Now</kbd>

:::note
Vulkan is the main supported graphics API for Magic Remote Rendering. Support for DirectX 11 and DirectX 12 are considered as experimental.
:::

### Alpha Channel in Unreal Engine 5

Magic Leap's Remote Rendering now supports alpha channels, which provides the user with segmented dimming and higher quality capture.

In order to take advantage of this feature in Unreal Engine OpenXR applications, you must first enable and control the output of the alpha channel.

First, you must prepare the application by allowing the engine to output alpha for postprocessing in the first place.

Go to **Project Settings** -> **Rendering** -> **Postprocessing** and set **Enable alpha channel support in post processing** to `Allow through tonemapper`.

![Enable Alpha Channel in Post-Processing](/img/unreal-5/alpha-channel/enable-alpha-channel-post-processing.png)

The alpha channel as exposed by Unreal Engine will not be enough for alpha blend layers in OpenXR, this is entirely due to the fact that the alpha output is inverted.

In order to correct this you will have to create a post-processing material that inverts the value of the alpha channel before it's submitted to the Remote Rendering service.

Go to **Window** and enable the **Content Browser**. From the **Content Browser**, add a new material in the desired directory.

![Content Browser New Material](/img/unreal-5/alpha-channel/content-browser-new-material.png)

The newly-created material must first be set up as a post-processing material with alpha support.

In the **Content Browser**, open the **Material Editor** by right-clicking on the material and selecting **Edit**.

Select the material in the **Material Graph** and under **Material** category set **Material Domain** to `Post Process`, then under the **Post Process Material** category enable `Output Alpha`.

![Material Graph Post-Process Material](/img/unreal-5/alpha-channel/material-graph-post-process-material.png)
![Material Graph Post-Process Material Output Alpha](/img/unreal-5/alpha-channel/material-graph-output-alpha.png)

Now you may proceed to creating the material, in this example alpha is being inverted directly from the post-processing input.

Add a new **SceneTexture** node and set **Scene Texture Id** to `PostProcessInput0`. This will be the main color input the material that will be worked on.

![Material Graph Scene Texture Node](/img/unreal-5/alpha-channel/material-graph-new-scene-texture.png)

The RGB channels will not be changing at all, so connect **SceneTexture.PostProcessInput0:Color** output to your material's **Emissive Color** input.

Add a **ComponentMask** node and a **OneMinus** node, then pipeline **SceneTexture.PostProcessInput0:Color** output through the **ComponentMask** node (set to `Alpha` only), then the **OneMinus** node, and finally into the material's **Opacity** input.

![Material Graph OneMinus](/img/unreal-5/alpha-channel/material-graph-component-mask-one-minus.png)

Add a **PostProcessVolume** actor in your level, enable `Infinite Extent (Unbound)` under its Post **Process Volume Settings** properties and add your material to **Post Process Materials** under **Rendering Features**.

![Level Post-Process Volume](/img/unreal-5/alpha-channel/level-post-process-volume.png)
![Post-Process Volume Infinite Extent](/img/unreal-5/alpha-channel/post-process-volume-infinite-extent.png)
![Level Post-Process Volume 2](/img/unreal-5/alpha-channel/material-graph-post-process-material-2.png)

## VR Preview

Once the editor has been restarted and shader compilation finishes, simply click <kbd>**VR Preview**</kbd>.

![VR Preview](/img/unreal-5/editor-vr-preview.png)

# Swapchains

Since we are going to draw to a window which is ultimately on the screen, things are a bit special.
We can't just "write" to the window's surface. Instead we have to go through what is called a
*swapchain*.

> **Note**: See also [the wikipedia article for a swap chain](https://en.wikipedia.org/wiki/Swap_Chain).

A swapchain is a group of one or multiple images, sometimes two images but most commonly three. If
you have ever heard terms such as *double buffering* or *triple buffering*, it refers to having
respectively two or three swapchain images.

The idea behind a swapchain is to draw to one of its images while another one of these images is
being shown on the screen. When we are done drawing we ask the swapchain to show the image we have
just drawn to, and in return the swapchain gives us drawing access to another of its images.

## Creating a swapchain

Swapchains have a lot of properties: the format and dimensions of their images, an optional
transformation, a presentation mode, and so on. We have to specify a value for each of these
parameters when we create the swapchain. Therefore before we can do so we have to query the
capabilities of the surface.

    let caps = window.surface().capabilities(physical)
        .expect("failed to get surface capabilities");

If we don't really care about all these properties, the only things that we need to choose is
the dimensions of the image (which have to be constrainted between a minimum and a maximum), the
behavior when it comes to transparency, and the format of the images.

    let dimensions = caps.current_extent.unwrap_or([1280, 1024]);
    let alpha = caps.supported_composite_alpha.iter().next().unwrap();
    let format = caps.supported_formats[0].0;

We can now create the swapchain:

    let (swapchain, images) = Swapchain::new(device.clone(), window.surface().clone(),
        caps.min_image_count, format, dimensions, 1, caps.supported_usage_flags, &queue,
        SurfaceTransform::Identity, alpha, PresentMode::Fifo, true, None)
        .expect("failed to create swapchain");
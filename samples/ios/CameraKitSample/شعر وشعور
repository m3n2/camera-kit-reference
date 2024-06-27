
import { bootstrapCameraKit, Transform2D } from "@snap/camera-kit";

window.addEventListener("load", async () => {
    try {
        // The JSON_WEB_TOKEN can be found in the SnapKit Portal, where it's called the ApiToken.
        //
        // Bootstrapping Camera Kit downloads the WebAssembly executable which contains the rendering engine.
        const cameraKit = await bootstrapCameraKit({ apiToken: JSON_WEB_TOKEN });

        // Creating the session initializes the rendering engine, and creates a CameraKitSession instance. The
        // CameraKitSession is used to interact with the rendering engine -- for example, setting an input video source
        // and applying Lenses.
        const session = await cameraKit.createSession();

        // When an error occurs, it means the current Lens could not be rendered. A real application will want to do
        // something more sophisticated here -- like asking the user to pick a different Lens, for example.
        session.events.addEventListener("error", (event) => console.error(event.detail));

        // We'll use the canvas-output element as a placeholder in our HTML, replacing it here with the <canvas> output
        // from CameraKitSession -- in particular, we're using the `live` output, which renders the Lens's Live
        // RenderTarget.
        //
        // (For more on RenderTargets, see [the LensStudio documentation](https://docs.snap.com/lens-studio/references/guides/lens-features/scene-set-up/camera#live-target-and-capture-target))
        document.getElementById("canvas-output")!.replaceWith(session.output.live);

        // We use the LensRepository to fetch a list of Lenses -- these are identified by a LensGroup ID. LensGroups
        // are configured in the Camera Kit Portal, where their IDs can be found.
        const { lenses } = await cameraKit.lensRepository.loadLensGroups([DEFAULT_GROUP_KEY]);

        // This section simply sets up the Lens <select> dropdown, populating it with our list of Lenses. When a lens
        // is chosen, we can apply it to the session.
        //
        // Only a single Lens can be applied at a time, so calling `applyLens()` always replaces any existing Lens
        // with the new one -- that is, there's no need to call `removeLens()` first.
        createLensSelect("lens-select", lenses, async (lens) => {
            await session.applyLens(lens)
        });

        // This section sets up the Camera Source <select> dropdown. It is populated with a list of video input devices
        // obtained from the browser, plus a Video and Image source.
        //
        // The most important parts are `session.setSource(source)` and `session.play('live')`, which have comments
        // below.
        //
        createSourceSelect(
            "source-select",
            async (source) => {
                try {
                    // The `source` here is a CameraKitSource, which can be created using a variety of helper methods
                    // provided by Camera Kit. For example, to create a source for a `MediaStream`, use the
                    // `createMediaStreamSource`.
                    await session.setSource(source);

                    // For webcams (our main use-case), it's natural to mirror the video -- this is easy to do with
                    // Camera Kit!
                    source.setTransform(Transform2D.MirrorX);

                    // Here we're adjusting the render size to improve user experience on mobile devices. This shows
                    // how render size can be set via the CameraKitSource.
                    //
                    // Keep in mind that setting render size *is not the same* as setting the size of the output on
                    // the webpage. Camera Kit render size sets the resolution at which Lenses are rendered -- it has a
                    // big impact on performance. When the CameraKitSession output <canvas> is added to the DOM, CSS can
                    // (and should) be used to set the size of the <canvas> on the page.
                    const shouldUsePortrait = parent.document.body.offsetWidth <= breakpoints.xs;
                    if (shouldUsePortrait) {
                        source.setRenderSize(480, 640);
                    }
                } catch (error) {
                    console.error(error);
                    throw error;
                }

                // Calling `play()` begins image processing. Now that we have a CameraKitSource attached to our
                // CameraKitSession, rendering will begin immediately -- without a Lens applied, this will simply
                // render the source to the output with no change (well, we are applying the MirrorX transformation, but
                // no other rendering happens without a Lens).
                //
                // Once a Lens is applied, we'll see the rendered Lens effect immediately now that we've already called
                // play().
                //
                // Note that we pass in "live" here -- this corresponds to the CameraKitSession.output.live <canvas>
                // element that we added to the DOM. ("live" is the default, but we've included it here for clarity.)
                //
                // CameraKitSession.pause() can be used to pause image processing, but we don't have a reason to do that
                // in this example.
                session.play("live");
            },
        );

    } catch (error: unknown) {
        console.error(error);
    }
});

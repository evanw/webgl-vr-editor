# WebGL VR Editor Experiment

This is an experimental voxel editor for [Google Cardboard](https://store.google.com/product/google_cardboard) and the iPhone.
Open www/index.html in Mobile Safari, rotate the phone 90 degrees to put it in the headset, and tap the headset button to use the current toolbar selection (the toolbar is at the bottom of the screen).

Live link: https://evanw.github.io/webgl-vr-editor/v1/

## Successes

* Camera movement

  Holding the headset button down to fly forward felt very natural.
  I was worried that flying instead of teleporting would induce motion sickness but that didn't happen for me.
  The constant velocity, restricted forward motion, and reticle in the center probably all helped.

* Tap to subtract

  This interaction ended up feeling very natural with the button on the headset.
  Sculpting felt fairly efficient even though it's only possible to edit one block at a time.

* Immersion

  Carving out a human-sized environment in close proximity around the camera felt very immersive.
  Distances felt very real and the lighting felt surprisingly believable even though it was low resolution.

* Temporal AA

  The previous frame is accumulated on to the current frame with a cycling sub-pixel jitter pattern and a rolling fade.
  The fade coefficient is zero if motion occurred or if the scene changed to avoid causing motion trails.
  Accumulating the previous frame during a yaw rotation isn't exactly correct because each eye camera has an offset left or right for stereo, but reducing the fade coefficient using the yaw velocity worked fine in practice.
  The reticle was drawn on top of the post-warped image to avoid motion trails (temporal AA accumulates the pre-warped image, not the post-warped image).

* Off-thread lightmap

  Voxel data edits are sent to a dedicated lightmap thread, which does a single-ray radiosity gather and posts the updated lightmap back to the main thread.
  This meant the main thread never had any framerate hiccups and I could afford to generate higher-quality lightmaps.
  The CPU on the iPhone 6 is also much faster than I expected and the lightmap computation delay was only a fraction of a second for the test scene.

* Auto-refresh development

  In debug mode, the main JavaScript file is repeatedly fetched and the page automatically refreshes when it changes.
  That made the app very convenient to develop because the iPhone didn't need to be removed from the headset.

  **Next time:**
  This would work even better if it was in an iframe because reloading the page in Mobile Safari always brings back the toolbar, which requires rotating the device to portrait and back to landscape to dismiss.
  Only do this if using WebGL inside an iframe doesn't cause any performance regressions.

## Failures

* Hover-based toolbar UI

  The tools are arranged in a toolbar around the user's feet.
  Having to look down every time you need to change tools was pretty irritating and slow.
  Another problem was that the UI was sometimes further away from the user than the geometry under it, which made it confusing to look at (the UI was always drawn on top of the ).

  **Next time:**
  Try a head tilt gesture to toggle the tool UI.
  Put the tool UI in world-space centered on the user's look vector at the time the UI was shown.

* Framerate problems

  Rendering was done by first rendering the scene twice to a texture, once for each eye, and then rendering that texture to the screen with a warp shader.
  This made it easy to do temporal AA but had a big impact on framerate.
  Single-pass rendering is consistently maxed out at 60fps on my iPhone 6 but two-pass rendering sometimes dropped down to 45fps.

  **Next time:**
  Do single-pass rendering using a mesh with a higher tessellation factor and apply the inverse lens distortion in the vertex shader.
  The "antialias" WebGL context creation flag has never worked in Mobile Safari unfortunately, so MSAA can't be used even though the hardware supports it.
  Try using geometry for anti-aliasing instead of doing temporal AA: find all mesh edges and render small 1px gradient skirts off the edges orthogonal to the camera.
  Only draw these for front faces and be careful about z-fighting.

* Lack of surface detail

  I didn't have a problem with this but several other people who tried this out said they had problems focusing on surfaces because the surfaces lacked any texture details.

  **Next time:**
  Add texture details.

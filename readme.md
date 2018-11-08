Tested on Unity 2018.2.2f1 64bit, Windows, Android 7.1.1
1. Add manomotion_sdk_2.0.0.2.unitypackage [link](https://www.manomotion.com/download-sdk-v2/)
2. Add manomotion_arCore_addon_2.unitypackage [link](https://www.manomotion.com/download-sdk-v2/)
3. Add AryzonSDK.unitypackage [link](https://s3.amazonaws.com/aryzondev/original/1X/AryzonSDK.zip)
4. Add xr-9.0.13.670.unitypackage [link](https://releases.8thwall.com/xr/unity/download/release)
5. Go to ManoMotionARCore/Scripts folder and delete RoomScannerARCore.cs
6. Patch XRVideoController.cs:
```
    lastRotation = rotation;

+   // Adjust to match hand size
+   scaleFactor *= 2f;

    Matrix4x4 mWarp = Matrix4x4.identity;
```
7. (using XR video texture instead of second camera with XRVideoController) Patch ManoMotionManagerARCore.cs:
```
    private void InitializeManoMotionManagerARcore()
    {
        ScaleRenderTexture();
        frameTexture = new Texture2D(render_texture.width, render_texture.height);
        framePixels = new Color32[render_texture.width * render_texture.height];
+        rotated = new Color32 [render_texture.width * render_texture.height];
    }
    new void Update()
    {
+        Graphics.Blit (xrController.GetRealityRGBATexture (), render_texture);
        ProcessARCoreFrame ();
    }

...
+        // FIXME: This should rotate the texture
+        int count = 0, width = frameTexture.width, height = frameTexture.height;
+        for (int i = 0; i < width; i++) {
+            for (int j = 0; j < height; j++) {
+                rotated [count] = framePixels [j * width + i];
+                count++;
+            }
+        }
+
+        rotated.CopyTo(_pixels, 0);
-        framePixels.CopyTo(_pixels, 0);
    }

+    static Color32 [] rotated;
```
FIXME: This gives a rotated hand texture
8. Setup your app bundle id
9. Enter Manomotion and 8thwall API keys and make sure you are using a matching bundle id (XR -> App key settings; ManoMotionManagerARCore -> Serial_key)
10. Deploy, turn phone into landscape and put into Aryzon headset

Scene settings gist:
* Aryzon
 - "Other Transform" - First Person Camera (camera moved by 8thwall XRCameraController)
 - Enable Set Aryzon mode on Start
 - Other events => "On start Aryzon" add FirstPersonCamera - Camera.enabled - Off
 - Other events => "On stop Aryzon" add FirstPersonCamera - Camera.enabled - On
* Manomotion
 - Turn off Show_background_layer

Cameras:
1) First Person Camera (MainCamera) [Depth only, cull everything, depth -10]
2) FrameClone with XRVideoController writing to render texture used by Manomotion [Solid color, cull nothing, depth -20]

Note: you may also need to install Aryzon official app and calibrate your headset first.  
Then copy the calibration code, disable ManoCalibration script and paste it in the app if it asks for it.
Blender Image Tracking Tutorial
===============================

.. note::
   This tutorial describes how to track points in a video using Blender and export
   their positions to CSV files for downstream analysis.

Requirements
------------

Software and Files
~~~~~~~~~~~~~~~~~~

- Blender (download from the official Blender website).
- ``displacement_tracking.blend`` Blender file.
- ``exportCSV.py`` script.
- A folder named ``data`` (or custom export folder) to store CSV output.
- The video file to analyze.

.. tip::
   Keep all files (``.blend``, ``.py``, and your video) in a clearly organized
   project directory so paths are easy to manage.

Step 0 – Setup
--------------

1. Install Blender and verify it runs on your system.
2. Place the following in your working folder:

   - ``displacement_tracking.blend``
   - ``exportCSV.py``
   - Your video file
   - A subfolder named ``data`` (for CSV exports)

3. Open Blender and confirm that you can load the ``displacement_tracking.blend`` file.

Step 1 – Tracking Points in Blender
-----------------------------------

1. Open Blender and load the project file:

   - ``File → Open`` → select ``displacement_tracking.blend``.

2. Switch to the *Movie Clip Editor* and load your video:

   - Click **Open** and select your video.
   - The video should now appear in the editor.

3. Define the frame range:

   - If you only want to analyze part of the video, adjust **Start** and **End** frames.
   - Use **Set Scene Frames** to sync the scene’s frame range to your selection.

4. Navigate the video:

   - Use the mouse scroll wheel to change zoom level.
   - Use the middle mouse button to pan/move the view.

5. Add tracking markers:

   - Use **Markers → Add** (or the equivalent “Add” button).
   - Click once on the point in the video you want a marker to follow.
   - After placing it, you can reposition the marker while it is selected
     (the square should appear white).

6. Track the marker over time:

   - With the marker selected, choose **Track → Track Forward**.
   - Wait for Blender to iterate through all frames.

7. Repeat as needed:

   - Rewind to the start of the video.
   - Add and track additional markers, one at a time.
   - Adjust tracking settings if more robust tracking is needed.

Step 2 – Exporting Displacements to CSV
---------------------------------------

1. Once all desired markers are tracked, switch to the **Scripting** workspace
   at the top of the Blender window.

2. Locate or open the text editor inside Blender’s Scripting workspace.

3. Load the export script:

   - Either copy–paste the contents of ``exportCSV.py`` into a new text block, or
   - Use **Open** in the text editor to load the existing ``exportCSV.py`` file.

4. Ensure there is a folder named ``data`` in your current working directory:

   - By default, the script writes CSV files into a ``data/`` subfolder.

5. Run the script:

   - In the Blender text editor, click **Run Script**.
   - The script will loop over all movie clips and tracks and create one CSV per track.

6. Confirm that output files were created:

   - Each file is named like ``tracking_<clipname>_<trackname>.csv``.
   - Columns are: **X, Y, frame_number**, where X and Y are in **pixel coordinates**.

Python Scripts
--------------

CSV Export Script (``exportCSV.py``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The script below iterates through all loaded movie clips and tracking tracks, and
exports marker positions as CSV files in the ``data`` folder.

.. code-block:: python

   import bpy
   D = bpy.data

   for clip in D.movieclips:
       for track in clip.tracking.tracks:
           fn = 'data/tracking_{0}_{1}.csv'.format(
               clip.name.split('.')[0],
               track.name
           )
           with open(fn, 'w') as f:
               frameno = 0
               while True:
                   markerAtFrame = track.markers.find_frame(frameno)
                   if not markerAtFrame:
                       break
                   frameno += 1
                   coords = markerAtFrame.co.xy
                   # Convert normalized coordinates to pixels
                   x_px = coords[0] * clip.size[0]
                   y_px = coords[1] * clip.size[1]
                   f.write('{0},{1},{2}\n'.format(x_px, y_px, frameno))
           f.close()

Optional Object Cleanup Script
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following snippet shows how to deselect all objects and delete the camera
object from the scene (example of scripting basic scene operations):

.. code-block:: python

   import bpy

   # Deselect all objects
   bpy.ops.object.select_all(action='DESELECT')

   # Select the camera (Blender 2.7x syntax)
   bpy.data.objects['Camera'].select = True

   # Delete the selected object(s)
   bpy.ops.object.delete()

Troubleshooting
---------------

CSV Export Produces More Files Than Expected
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem:**  
The export script outputs more CSV files than the number of tracking points you
created for your current video.

**Likely Cause:**  
Movie clips or tracking data from previous sessions are still loaded in Blender.

**Solution:**

1. List all loaded movie clips in the Python console:

   .. code-block:: python

      import bpy
      for clip in bpy.data.movieclips:
          print(clip.name)

2. Remove unwanted clips by name:

   .. code-block:: python

      import bpy
      bpy.data.movieclips.remove(
          bpy.data.movieclips["your_clip_name"]
      )

3. Re-run the ``exportCSV.py`` script after cleaning up the unused clips.

Notes
-----

- Always verify that your **frame range** (Start/End) matches the video portion
  you intend to analyze.
- If tracking fails or jumps, consider improving contrast, adjusting tracking
  settings, or manually correcting markers before exporting.
- The exported CSV files are suitable for downstream processing in Python,
  MATLAB, Excel, or other analysis tools.

Download Files
--------------

- `displacement_tracking.blend <files/image-analysis/displacement_tracking.blend>`__
- `exportCSV.py <files/image-analysis/exportCSV.py>`__



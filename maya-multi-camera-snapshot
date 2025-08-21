import maya.cmds as cmds
import os
import datetime

def get_all_cameras():
    """
    Get list of all cameras in the scene
    """
    all_cameras = []
    # Find all camera shapes
    camera_shapes = cmds.ls(type='camera')
    
    for cam_shape in camera_shapes:
        # Get transform node of the camera
        cam_transforms = cmds.listRelatives(cam_shape, parent=True, type='transform')
        if cam_transforms:
            cam_name = cam_transforms[0]
            # Filter out system cameras (persp, top, front, side)
            if cam_name not in ['persp', 'top', 'front', 'side']:
                all_cameras.append(cam_name)
    
    return all_cameras

def snap_selected_cameras(selected_cameras):
    """
    Snap images from selected cameras
    """
    
    if not selected_cameras:
        cmds.warning("Please select at least 1 camera!")
        return
    
    print(f"📷 Selected cameras: {selected_cameras}")
    
    # Check existing cameras
    existing_cameras = []
    for cam in selected_cameras:
        if cmds.objExists(cam):
            existing_cameras.append(cam)
        else:
            print(f"⚠️ Warning: Camera '{cam}' not found in scene")
    
    if not existing_cameras:
        cmds.warning("No cameras found in the list!")
        return
    
    # Get render resolution from render settings
    width = cmds.getAttr('defaultResolution.width')
    height = cmds.getAttr('defaultResolution.height')
    
    print(f"📏 Resolution: {width} x {height}")
    
    # Create folder for images (if doesn't exist)
    project_path = cmds.workspace(q=True, rd=True)
    images_folder = os.path.join(project_path, "images", "snapshots")
    
    if not os.path.exists(images_folder):
        os.makedirs(images_folder)
        print(f"📁 Created folder: {images_folder}")
    
    # Current date for timestamp (no time)
    date_stamp = datetime.datetime.now().strftime("%Y%m%d")
    
    # Get namespace from reference files
    all_namespaces = cmds.namespaceInfo(listNamespace=True)
    # Filter namespaces from references (exclude UI, shared)
    ref_namespaces = [ns for ns in all_namespaces if ns not in ['UI', 'shared']]
    
    if ref_namespaces:
        # Use first namespace found, or let user choose if multiple
        if len(ref_namespaces) == 1:
            namespace_name = ref_namespaces[0]
        else:
            # If multiple namespaces, show list and use first one
            print(f"🎭 Found multiple namespaces: {ref_namespaces}")
            namespace_name = ref_namespaces[0]  # Use first one
            print(f"📝 Using namespace: {namespace_name}")
    else:
        namespace_name = "no_reference"  # If no reference
    
    print(f"🎭 Namespace: {namespace_name}")
    
    # Store current camera to restore after completion
    current_panel = cmds.getPanel(wf=True)
    if cmds.getPanel(to=current_panel) == 'modelPanel':
        original_camera = cmds.modelPanel(current_panel, q=True, cam=True)
    else:
        # If active panel is not model panel, find first model panel
        model_panels = cmds.getPanel(type='modelPanel')
        if model_panels:
            current_panel = model_panels[0]
            original_camera = cmds.modelPanel(current_panel, q=True, cam=True)
        else:
            cmds.warning("No suitable model panel found!")
            return
    
    snapped_files = []
    
    try:
        for cam in existing_cameras:
            print(f"📸 Snapping from camera: {cam}")
            
            # Switch camera in viewport
            cmds.modelPanel(current_panel, edit=True, cam=cam)
            
            # Set filename: camera_namespace_date
            filename = f"{cam}_{namespace_name}_{date_stamp}"
            filepath = os.path.join(images_folder, filename)
            
            # Delete existing file if exists to avoid overwrite error
            full_filepath = filepath + ".png"
            if os.path.exists(full_filepath):
                try:
                    os.remove(full_filepath)
                    print(f"🗑️ Removed existing file: {filename}.png")
                except:
                    print(f"⚠️ Could not remove existing file: {filename}.png")
            
            # Snap image with playblast (fast and accurate)
            result = cmds.playblast(
                frame=cmds.currentTime(q=True),  # Current frame
                format='image',
                compression='png',
                quality=100,
                width=width,
                height=height,
                percent=100,
                viewer=False,  # Don't open viewer
                showOrnaments=True,  # Show grid, axis if any
                filename=filepath,
                completeFilename=full_filepath,
                clearCache=True,
                forceOverwrite=True  # Force overwrite to avoid error
            )
            
            if result:
                snapped_files.append(full_filepath)
                print(f"✅ Saved: {filename}.png")
            else:
                print(f"❌ Could not snap from camera {cam}")
        
        # Restore original camera
        cmds.modelPanel(current_panel, edit=True, cam=original_camera)
        
        # Summary results
        print(f"\n🎉 Snap completed! Total {len(snapped_files)} images")
        print(f"🎭 Namespace: {namespace_name}")
        print(f"📅 Date: {date_stamp}")
        print(f"📂 File location: {images_folder}")
        
        for filepath in snapped_files:
            print(f"   - {os.path.basename(filepath)}")
            
        # Open folder (Windows/Mac)
        if os.name == 'nt':  # Windows
            os.startfile(images_folder)
        elif os.name == 'posix':  # Mac/Linux
            os.system(f'open "{images_folder}"')
            
    except Exception as e:
        print(f"❌ Error occurred: {str(e)}")
        # Restore original camera even if error occurs
        cmds.modelPanel(current_panel, edit=True, cam=original_camera)

def select_all_cameras(camera_checkboxes, value):
    """
    Select/deselect all cameras
    """
    for checkbox in camera_checkboxes.values():
        cmds.checkBox(checkbox, edit=True, value=value)

def snap_from_ui(camera_checkboxes):
    """
    Snap images from cameras selected in UI
    """
    selected_cameras = []
    
    for cam_name, checkbox in camera_checkboxes.items():
        if cmds.checkBox(checkbox, query=True, value=True):
            selected_cameras.append(cam_name)
    
    if selected_cameras:
        snap_selected_cameras(selected_cameras)
    else:
        cmds.warning("Please select at least 1 camera!")

def create_snapshot_button():
    """
    Create UI for selecting cameras and snapping images
    """
    
    # Delete old window if exists
    window_name = "multiCamSnapshotWindow"
    if cmds.window(window_name, exists=True):
        cmds.deleteUI(window_name)
    
    # Get list of all cameras
    available_cameras = get_all_cameras()
    
    if not available_cameras:
        cmds.confirmDialog(
            title="No Cameras Found",
            message="No cameras found in scene. Please create cameras first.",
            button=["OK"]
        )
        return
    
    # Create new window
    window = cmds.window(
        window_name,
        title="🌸 Emily's Multi-Camera Snapshot",
        widthHeight=(400, 300),
        resizeToFitChildren=True,
        sizeable=True
    )
    
    # Main layout
    main_layout = cmds.columnLayout(adjustableColumn=True, columnOffset=('both', 20))
    
    cmds.text(
        label="📸 Select Cameras to Snap",
        font="boldLabelFont",
        height=40
    )
    
    cmds.text(
        label=f"Found {len(available_cameras)} cameras (using Namespace)",
        font="plainLabelFont",
        height=30
    )
    
    cmds.separator(height=15)
    
    # Create scroll layout for checkboxes
    scroll_layout = cmds.scrollLayout(height=150, childResizable=True)
    
    # Store checkbox references
    camera_checkboxes = {}
    
    for cam in available_cameras:
        # Create checkbox for each camera
        checkbox = cmds.checkBox(
            label=f"📷 {cam}",
            value=True,  # Select all by default
            height=30
        )
        camera_checkboxes[cam] = checkbox
    
    # Back to main layout
    cmds.setParent(main_layout)
    
    cmds.separator(height=15)
    
    # Control buttons
    cmds.rowLayout(numberOfColumns=3, columnWidth3=(120, 120, 120))
    
    cmds.button(
        label="✅ Select All",
        height=35,
        command=lambda x: select_all_cameras(camera_checkboxes, True)
    )
    
    cmds.button(
        label="❌ Deselect All",
        height=35,
        command=lambda x: select_all_cameras(camera_checkboxes, False)
    )
    
    cmds.button(
        label="✨ Snap!",
        height=35,
        backgroundColor=[0.8, 0.9, 1.0],
        command=lambda x: snap_from_ui(camera_checkboxes)
    )
    
    cmds.setParent(main_layout)
    
    cmds.separator(height=10)
    
    cmds.text(
        label="Resolution from Render Settings automatically 💫",
        font="plainLabelFont",
        height=30
    )
    
    cmds.showWindow(window)

# Main function call
if __name__ == "__main__":
    # Create UI
    create_snapshot_button()
    
    # Or call directly with desired cameras
    # snap_selected_cameras(['camera1', 'camera2', 'camera3'])

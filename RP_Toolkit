import nuke
import os
import platform
import subprocess
import nuke.rotopaint as rp
import traceback



def upstream() :
    nodes = nuke.selectedNode()
    upstream_nodes = nodes.dependencies()
    if nodes.Class() == 'Roto' :
        if upstream_nodes :
            if upstream_nodes[0].Class() == 'CornerPin2D' :
                pass
            else :
                before_node = upstream_nodes[0]
                nuke.delete(before_node)

def getPath():
    node = nuke.selectedNodes()
    try :
        if node :
            selected_node = node[0]
            if selected_node.Class() == 'Viewer' :
                return os.path.dirname(nuke.root().name())
            elif 'file' in selected_node.knobs():
                node_filePath = selected_node['file'].getValue()
                if os.path.isdir(node_filePath):
                    return node_filePath
                return os.path.dirname(node_filePath)
        return os.path.dirname(nuke.root().name())
    except Exception as e:
        nuke.message("Error on accessing path : {}".format(e))
    return 0


def OpenDirectory(): 
    dir_path = getPath()   
    if dir_path :
        if platform.system() == 'Windows':
            os.startfile(dir_path )
        else :
            subprocess.Popen(['caja', dir_path ])  
    else : nuke.message("Couldn't open File Browser")
    return 0


def autocrop() :
    userName = os.getlogin()
    n = nuke.createNode('NoOp')
    x, y = (n.xpos(), n.ypos())
    nuke.delete(n)
    path = "/usr/people/"+userName+"/.nuke/RP_Toolkit/AutoCrop.nk"
    nuke.scriptReadFile(path)
    for node in nuke.selectedNodes() :
        node.setXYpos(x, y)

def dasgrainPro() :
    userName = os.getlogin()
    n = nuke.createNode('NoOp')
    x, y = (n.xpos(), n.ypos())
    nuke.delete(n)
    path = "/usr/people/"+userName+"/.nuke/RP_Toolkit/dasgrainPro.nk"
    nuke.scriptReadFile(path)
    for node in nuke.selectedNodes() :
        node.setXYpos(x, y)

def cornerPin_To_Tracker() :

    frame = first_frame = int(nuke.Root()['first_frame'].getValue())
    last_frame = int(nuke.Root()['last_frame'].getValue())
    node = nuke.selectedNodes()
   
    if node and node[0].Class() == 'CornerPin2D' :

        cornerPin = node[0] 

        tracker = nuke.createNode('Tracker4')
        for i in range(4) :
            tracker['add_track'].execute()

        tracker['reference_frame'].setValue(nuke.frame())

        for frame in range(first_frame, last_frame+1 ) : 
            nuke.frame(frame)

            to1x = cornerPin['to1'].valueAt(frame)[0]
            to1y = cornerPin['to1'].valueAt(frame)[1]
            to2x = cornerPin['to2'].valueAt(frame)[0]
            to2y = cornerPin['to2'].valueAt(frame)[1]
            to3x = cornerPin['to3'].valueAt(frame)[0]
            to3y = cornerPin['to3'].valueAt(frame)[1]
            to4x = cornerPin['to4'].valueAt(frame)[0]
            to4y = cornerPin['to4'].valueAt(frame)[1]
            tracker['tracks'].setValueAt(to1x, frame, 31*0+2)
            tracker['tracks'].setValueAt(to1y, frame, 31*0+3)
            tracker['tracks'].setValueAt(to2x, frame, 31*1+2)
            tracker['tracks'].setValueAt(to2y, frame, 31*1+3)
            tracker['tracks'].setValueAt(to3x, frame, 31*2+2)
            tracker['tracks'].setValueAt(to3y, frame, 31*2+3)
            tracker['tracks'].setValueAt(to4x, frame, 31*3+2)
            tracker['tracks'].setValueAt(to4y, frame, 31*3+3)
        for i in range(4):
            tracker['tracks'].setValue(True, 31*i+6)   
            tracker['tracks'].setValue(True, 31*i+7) 
            tracker['tracks'].setValue(True, 31*i+8) 
        tracker['transform'].setValue('match-move')
        tracker['label'].setValue(cornerPin['name'].getValue())
    else :
        nuke.message("Please Select CornerPin Node")

def tracker_to_Roto() :
    first_frame = int(nuke.root()['first_frame'].getValue())
    last_frame = int(nuke.root()['last_frame'].getValue())
    
    selected_node = nuke.selectedNodes()
    
    if selected_node :
        try :
            if selected_node[0].Class() == 'Tracker4' :       
               tracker = selected_node[0]      
               tracker['translate'].valueAt(50)
               roto = nuke.createNode('Roto')
               curve = roto['curves']
               rootLayer = curve.rootLayer
               new_layer = rp.Layer(curve)
               rootLayer.append(new_layer)
               
               layer = curve.toElement('Layer1')
               name = tracker['name'].getValue()
               layer.name = "Layer1(" + name +")"
               
               for frame in range(first_frame, last_frame +1):
                   translate_X = tracker['translate'].getValueAt(frame , 0)
                   translate_Y = tracker['translate'].getValueAt(frame , 1)
                   rotate = tracker['rotate'].getValueAt(frame)
                   scale = tracker['scale'].getValueAt(frame)
                   center_X = tracker['center'].getValueAt(frame , 0)
                   center_Y = tracker['center'].getValueAt(frame , 1)
        
                   transform = layer.getTransform()
                   translate_X_curve = transform.getTranslationAnimCurve(0)
                   translate_Y_curve = transform.getTranslationAnimCurve(1)
                   scale_X_curve = transform.getScaleAnimCurve(0)
                   scale_Y_curve = transform.getScaleAnimCurve(1)
                   rotation_curve = transform.getRotationAnimCurve(2)
                   pivot_X_curve = transform.getPivotPointAnimCurve(0) 
                   pivot_Y_curve = transform.getPivotPointAnimCurve(1)
        
                   if translate_X_curve :
                        translate_X_curve.addKey(frame, translate_X)        
                   if translate_Y_curve :
                        translate_Y_curve.addKey(frame, translate_Y)        
                   if scale_X_curve :
                        scale_X_curve.addKey(frame, scale)        
                   if scale_Y_curve :
                        scale_Y_curve.addKey(frame, scale)        
                   if rotation_curve :
                        rotation_curve.addKey(frame, rotate)        
                   if pivot_X_curve :
                        pivot_X_curve.addKey(frame, center_X)        
                   if pivot_Y_curve :
                        pivot_Y_curve.addKey(frame, center_Y)
                       
            else :
               nuke.message("Please select Tracker Node")
        
        except Exception as e :
            nuke.message("Error message : {}".format(e))
            traceback.print_exc()
    
    else :
       nuke.message("Please select Tracker Node")

def roto_shape_finder(root_layer = None, roto_shapes =[]):
    if not root_layer:
        root_layer = curves.rootLayer
    for roto_item in root_layer:
        if isinstance(roto_item, rp.Shape):
            roto_shapes.append(roto_item)
        if isinstance(roto_item, rp.Layer):
            roto_shape_finder(roto_item, roto_shapes)

def rotoShapeToTracker() :
    node = nuke.selectedNode()
    first_frame = int(nuke.root()['first_frame'].getValue())
    last_frame = int(nuke.root()['last_frame'].getValue())
    current_frame = nuke.frame()

    C = 31
    X = 2
    Y = 3
    T = 6
    R = 7
    S = 8

    if node.Class() == 'Roto' :
        curves = node['curves']
        root_layer = curves.rootLayer
        shapes = [] 
        shape_name = []       
        roto_shape_finder(root_layer=root_layer,roto_shapes=shapes)
        for shape in shapes :
            name = shape.name.strip()
            shape_name.append(name)
        p = nuke.Panel('custom one')
	p.addBooleanCheckBox('Delete extra shapes/points. Max 4 points are allowed.', True)
        p.addEnumerationPulldown('Select Shape to Export', ' '.join(shape_name))
        if p.show() :
            selection = str(p.value('Select Shape to Export'))
            tracker = nuke.createNode('Tracker4')
            for i in range(4):
                tracker['add_track'].execute()
            for frame in range(first_frame, last_frame + 1):
                for obj in shapes:
                    if obj.name == selection:
                        for i, point in enumerate(obj):
                            if i < 4:
                                pos = point.center.getPosition(frame)
                                tracker['tracks'].setValueAt(int(pos[0]), frame, C*i+X)
                                tracker['tracks'].setValueAt(int(pos[1]), frame, C*i+Y)
                            else :
                                break
            for i in range(4):
                tracker['tracks'].setValue(True, C*i+T)
                tracker['tracks'].setValue(True, C*i+R)
                tracker['tracks'].setValue(True, C*i+S)
            tracker['transform'].setValue(3)
            tracker['reference_frame'].setValue(current_frame)
            tracker['label'].setValue(node['name'].getValue())
        
    else :
        nuke.message("Please select Roto node")


def cornerPin_Export():
    p = nuke.Panel('cornerPin_Export')
    p.addEnumerationPulldown('Export CornerPin to', 'Tracker Roto')
    n = nuke.selectedNodes()
    if n :
        if n[0].Class() == 'CornerPin2D':
            if p.show() :
                selection = p.value('Export CornerPin to')
                if selection == 'Tracker' :
                    cornerPin_To_Tracker()
            
                elif selection == 'Roto' :
                    cornerPin_To_Tracker()
                    tracker_to_Roto()

        else :
            nuke.message("Please select CornerPin Node")
    else :
        nuke.message("Please select CornerPin")
    upstream()


def note_creator():    
    Plate = nuke.selectedNode()
    if Plate:
        file_path = Plate['file'].getValue()
        shot_dir_path = os.path.dirname(file_path)
    else :
        nuke.message("nothing selected")   
    script_path = nuke.root().name()
    s = nuke.createNode('StickyNote')
    s['label'].setValue("Hi\n\nCleanup Done as per reference provided\n\nRender Path : "+ shot_dir_path + "\n\nScript Path : " + script_path + "\n\nThankYou.")


def script_save():
    
    selected_node = nuke.selectedNodes()
 
    if selected_node:

        first_frame = selected_node[0].knob('first').value()
        last_frame = selected_node[0].knob('last').value()
        format = selected_node[0].knob('format').value()    

        nuke.root()['first_frame'].setValue(first_frame)
        nuke.root()['last_frame'].setValue(last_frame)
        nuke.root()['format'].setValue(format)

        for node in selected_node:
            shot_path = node['file'].value()
            dir_path = os.path.dirname(node['file'].value())
            
 
        while os.path.basename(dir_path) != "elements":
            dir_path = os.path.dirname(dir_path)
                
        path = os.path.dirname(dir_path)
          
        folder_struct = ["prep"]
    
        #for i in range (1):
        path = os.path.join(path, folder_struct[0])

        finalFolder_path = path + "/Final"

        if not os.path.exists(finalFolder_path) :
            os.mkdir(finalFolder_path)            

        username = os.getlogin()       
    
        path = os.path.join(path, username)
           
        if not os.path.exists(path):
            os.mkdir(path)                
    
        path = os.path.join(path, "Nuke")
        if not os.path.exists(path):
            os.mkdir(path)
            
    
        shot_path = shot_path.rsplit(".",2)[0]
        basename = os.path.basename(shot_path)

    
        parts = basename.split("_")
        
        parts[0] = "f"

        parts.insert(-2, 'prep')
        
    
        new_basename = "_".join(parts) + "_00"   
        
        files = os.listdir(path)
        sorted_files = sorted(files)
        
        
        filetered_sorted_list = [file_name for file_name in sorted_files if "AutoSave" not in file_name and not file_name.endswith("~")]

        if filetered_sorted_list:
            last_element =    filetered_sorted_list[-1]   
                    
            match = re.search(r'(\d+)(?=\.\w+$)', last_element)
            if match:
                number_str = match.group(1)
                number = int(number_str)
                
                incremented_number = number + 1
                
                incremented_number_str = str(incremented_number).zfill(len(number_str))
                
                new_string_name = re.sub(r'\d+(?=\.\w+$)', incremented_number_str, last_element)
                   
        
            final_path = os.path.join(path, new_string_name)
            

        else : 
            file_name = os.path.basename(shot_path)
            shotName = file_name.split('.%04d.')[0] + "_001.nk"
            parts = shotName.split("_")
            parts[0] = "f"
            parts.insert(-2, 'prep')
            new_shot_name = "_".join(parts)

            final_path = os.path.join(path, new_shot_name)
                            
        nuke.scriptSaveAs(final_path)
            
    else: nuke.message("Please select a Scan which you would like to save it.")


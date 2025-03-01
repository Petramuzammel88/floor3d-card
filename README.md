# floor3d-card
Javascript Module for the Home Assistant visualization Card for 3D Models with bindings to entity states. Be advised it is still an alfa version; it requires a bit of manual installation actions and related troubleshooting. I'm working at a more official and easy to setup version using card templates projects.

|Demo [![Alt text](https://img.youtube.com/vi/M1zlIneB3e0/0.jpg)](https://www.youtube.com/watch?v=M1zlIneB3e0)   | Tutorial [![Alt text](https://img.youtube.com/vi/RVDNxt2tyhY/0.jpg)](https://www.youtube.com/watch?v=RVDNxt2tyhY)  |
|---|---|

## Installation

**Pay attention the old config need some changes. I recommmend to save the card config from a previous version prior to upgrade to the new version. Then apply changes to your saved config in a normal editor and put back the new config in  a brand new card; Apologies for any incovenience**

Waiting for the approval of the card in the HACS default repositories [Home Assistant Community Store](https://github.com/hacs/integration), you can include this repository (https://github.com/adizanni/floor3d-card) into the HACS custom repository and start using it.

You can also  download the compiled js file from here (https://raw.githubusercontent.com/adizanni/floor3d-card/master/dist/floor3d-card.js) and upload it to your www home assistant folder

It's **required** to load this card as `module`.

```yaml
- url: /local/pathtofile/floor3d-card.js
  type: module
```

## Model Design and Installation
Use a 3D modeling software. As you have to model your home I would suggest this software (the one I tested): http://www.sweethome3d.com/.
Model your home with all needed objects and furniture (I will post here some hints on how to better design your home for best results with the custom card).
For further instruction I assume you will use SweetHome3D.
At the end of your modeling, you need to export the files in obj format using '3D View \ Export to OBJ format ...', specify the folder where you want to store the output (be careful there are multiple files)
Copy the full set of files (minimum is the .obj file and .mtl file) to a sub folder of /config/www in Home assistant.
Be aware that when you remove objects from the model the object ids get reassigned: This meanse that after a modification and re-export of your model it is possible you need to redo the bindings with the new object names. I'm trying to find solutions or workaround, but it is not an easy task.It could be a good practice to make the objects invisble instead of removing them (not yet tested if this solution preserves the objects ids).
Then configure a new panel card with the following options:

## Options

| Name | Type | Default | Description
| ---- | ---- | ------- | -----------
| type | string | **Required** | `custom:floor3d-card`.
| name | string | Floor 3d | the name of the card.
| entities | array | none | list of enitities to bind to 3D model objects.
| style | string | none | the style that will be applied to the canvas element of the card.
| path | string | **Required** | path to the Waterforont obj (objects), mtl (material) and other files.
| objfile | string | **Required** | object file name (.obj) Waterfront format.
| mtlfile | string | **Required** | material file name (.mtl) Waterfront format.
| backgroundColor | string | '#aaaaaa' | canvas background color
| globalLightPower  | float | 0.3 | intensity of the light illuminating the full scene it can also the name of a numeric sensor


For each enity in the entities list you need to specify the following options:

| Name | Type | Default | Description
| ---- | ---- | ------- | -----------
| entity | string | **Required** | your entity id.
| object_id | string | **Required** | the name of the object in the model to biind to your entity.
| type3d | string | **Required** | the type of object binding. Values are: light, hide, color, text, gesture

**Note: to facilitate the configuration you can load the model without entity bindings and you will be able to show the object_id you want to bind to by double clicking on the object**

## Camera Rotation and Camera Position

For **camera rotation and position** recording config:
```yaml
camera_position:
  x: <x coordinate of the recorded camera positioon>
  y: <y coordinate of the recorded camera positioon>
  z: <z coordinate of the recorded camera positioon>
camera_rotate:
  x: <x coordinate of the recorded camera rotation>
  y: <y coordinate of the recorded camera rotation>
  z: <z coordinate of the recorded camera rotation>
```
When in edit mode you can double click in an empty model space to retrieve the current postition and rotation of the camera. You can retrieve the 2 sets of coordinates from the prompt box that will appear. You can then manually copy the content and paste to the card config in code editor mode. Thanks to this the new default position of the camera will be set to the configured coordinates. 

## Lights

For **light** example config:
```yaml
entities:
  - entity: <a light entity id>
    type3d: light
    object_id: <an object id in the 3D model you want to postion the light on>
    light:
      lumens: <max light lumens range: 0-4000 for regular led/bulb lights>
```

light_name is the name of the light object that will be created in the model to do the actual illumination.

Light behaviour is obvious: the **light_name** will illuminate when the bound entity in Home Assistant will be turned on and viceversa.
A double click on the light object will toggle the light (so far the events in iOS and Android are not yet managed as the events are captured by the OrbitContol of Three.js library and I have not yet fully understood the behaviour)

## Hide

For **hide** example config:
```yaml
entities
  - entity: <a binary sensor entity id>
    type3d: hide
    object_id: <an object_id in the model you want to hide if condition is true>
    hide:
      state: <the state of the entity triggering the hiding of the object: ex 'off'>
```

Hide behavour: the object_id will be hidden when the state of the bound entity will be equal to the **state** value

## Show

For **show** example config:
```yaml
entities
  - entity: <a binary sensor entity id>
    type3d: hide
    object_id: <an object_id in the model you want to hide if condition is true>
    show:
      state: <the state of the entity triggering the showing of the object: ex 'off'>
```

Show behavour: the object_id will be visible when the state of the bound entity will be equal to the **state** value

## Color

For **color** example config:
```yaml
entities:
  - entity: <a discrete sensor entity id>
    type3d: color
    object_id: <the object id in the 3D model that has to change color based on the state of the entity>
    colorcondition:
      - color: <color to paint if condition for the entity id in the stat to be true ex:'#00ff00'>
        state: <state of the entity>
      .......
```

Color behavour: the object_id will be painted in the color when the state of the bound entity will be equal to the **state** value

## Text

For **text** example config:
```yaml
entities:
  - entity: <a numeric or text sensor entity id>
    type3d: text
    object_id: <the plane object id in the 3D model that will allow the display of the state text>
    text:
      - font: <name of the font text ex:'verdana'>
        textbgcolor: <background color for the text. ex: '#000000' or 'black'>
        textfgcolor: <foreground color for the text. ex: '#ffffff' or 'white'>
      .......
```

Text behaviour: the object_id representing the plane object (ex. mirror; picture, tv screen, etc) will display the state text for the entity

## Gesture

For **gesture** (action) example config:
```yaml
entities:
  - entity: <an actionable entity>
    type3d: gesture
    object_id: <an object id in the 3D model you want to double click to trigger the gesture/action>
    gesture:
      domain: <the domain of the service to call>
      service: <the service to call>
```
when you double click on the object, the domain.service is called with data { entity_id: entity }
(so far the iOS and Android events are not yet managed as the events are captured by the OrbitContol of Three.js library and I have not yet fully understood the behaviour)

### Example

To give it a try please, load the example folder files in a folder within /config/www of your Home Assistant.
Create a new Panel View add Floor3d-card and cut and paste the following config:

```yaml
type: 'custom:floor3d-card'
entities:
  - entity: <your light entity id>
    type3d: light
    object_id: sweethome3d_opening_on_hinge_2_LampSide_31
    light:
      lumens: 500
  - entity: <your binary sensor entity id (example a magnet sensor for a window)>
    type3d: color
    object_id: sweethome3d_window_pane_on_hinge_1_50
    colorcondition:
      - state: 'on'
        color: '#00ff00'
      - state: 'off'
        color: '#ff0000'
path: /local/home2/
objfile: MyExampleHome2.obj
mtlfile: MyExampleHome2.mtl
backgroundColor: '#000001'
globalLightPower: 0.4
```

<img width="300" alt="Example Light On" src="https://github.com/adizanni/visualization-card-3d-floor/blob/main/images/ExampleOn.png?raw=true">
<img width="300" alt="Example Light Off" src="https://github.com/adizanni/visualization-card-3d-floor/blob/main/images/ExampleOff.png?raw=true">

### To Do
List of feature I will develop in the next releases:
- Condition 3D Type: add templating and or support complex conditions

Project General Availability (https://github.com/adizanni/floor3d-card/projects/1)




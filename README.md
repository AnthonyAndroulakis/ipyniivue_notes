ipyniivue is a Jupyter widget that wraps Niivue, a lightweight JavaScript library for 3D rendering of neuroimaging data. It allows users to embed interactive neuroimaging visualizations in Jupyter notebooks, supporting NIfTI images and various neuroimaging data formats.

---

## Contents

1. [Enumerations](#enumerations)
   - [SliceType](#slicetype)
   - [DragMode](#dragmode)
   - [MultiplanarType](#multiplanartype)
2. [Classes](#classes)
   - [NiiVue](#niivue)
     - [Initialization](#initialization)
     - [Methods](#methods)
     - [Properties](#properties)
   - [Volume](#volume)
   - [Mesh](#mesh)
   - [WidgetObserver](#widgetobserver)

---

## Enumerations

### SliceType

An enumeration representing different slice types for visualization.

```python
from ipyniivue import SliceType
```

- `SliceType.AXIAL` (0)
- `SliceType.CORONAL` (1)
- `SliceType.SAGITTAL` (2)
- `SliceType.MULTIPLANAR` (3)
- `SliceType.RENDER` (4)

### DragMode

An enumeration representing different drag modes.

```python
from ipyniivue import DragMode
```

- `DragMode.CONTRAST` (1)
- `DragMode.MEASUREMENT` (2)
- `DragMode.PAN` (3)

### MultiplanarType

An enumeration representing different multiplanar layout options.

```python
from ipyniivue import MultiplanarType
```

- `MultiplanarType.AUTO` (0)
- `MultiplanarType.COLUMN` (1)
- `MultiplanarType.GRID` (2)
- `MultiplanarType.ROW` (3)

---

## Classes

### NiiVue

The main class representing a Niivue widget in Jupyter notebooks.

#### Initialization

```python
NiiVue(height: int = 300, **options)
```

- **Parameters:**
  - `height` (_int_, optional): The height of the widget in pixels. Default is `300`.
  - `**options`: Additional keyword arguments representing various visualization options.

- **Description:** Initializes a new NiiVue widget with the given height and options.

#### Methods

##### load_volumes

```python
load_volumes(volumes: list)
```

- **Parameters:**
  - `volumes` (_list_): A list of dictionaries, where each dictionary contains information about a volume to be loaded.

- **Description:** Loads a list of volumes into the widget. Existing volumes are replaced.

- **Example:**

  ```python
  nv = NiiVue()
  nv.load_volumes([
      {
          'path': 'path/to/volume.nii',
          'colormap': 'gray',
          'opacity': 1.0,
      },
      # ... more volumes
  ])
  ```

---

##### add_volume

```python
add_volume(volume: dict)
```

- **Parameters:**
  - `volume` (_dict_): A dictionary containing information about a volume to be added.

- **Description:** Adds a single volume to the widget without replacing existing volumes.

- **Example:**

  ```python
  nv.add_volume({
      'path': 'path/to/another_volume.nii',
      'colormap': 'hot',
      'opacity': 0.5,
  })
  ```

---

##### load_meshes

```python
load_meshes(meshes: list)
```

- **Parameters:**
  - `meshes` (_list_): A list of dictionaries, where each dictionary contains information about a mesh to be loaded.

- **Description:** Loads a list of meshes into the widget. Existing meshes are replaced.

- **Example:**

  ```python
  nv.load_meshes([
      {
          'path': 'path/to/mesh.obj',
          'rgba255': [255, 255, 255, 255],
          'opacity': 1.0,
      },
      # ... more meshes
  ])
  ```

---

##### add_mesh

```python
add_mesh(mesh: dict)
```

- **Parameters:**
  - `mesh` (_dict_): A dictionary containing information about a mesh to be added.

- **Description:** Adds a single mesh to the widget without replacing existing meshes.

- **Example:**

  ```python
  nv.add_mesh({
      'path': 'path/to/another_mesh.obj',
      'rgba255': [0, 255, 0, 255],
      'opacity': 0.7,
  })
  ```

---

#### Properties

##### volumes

```python
@property
volumes -> list
```

- **Returns:** A list of `Volume` instances currently loaded in the widget.

- **Description:** Provides access to the list of volumes. Allows for modification of volume properties.

- **Example:**

  ```python
  # Get the first volume's opacity
  volume_opacity = nv.volumes[0].opacity

  # Set the first volume's opacity
  nv.volumes[0].opacity = 0.8
  ```

---

##### meshes

```python
@property
meshes -> list
```

- **Returns:** A list of `Mesh` instances currently loaded in the widget.

- **Description:** Provides access to the list of meshes. Allows for modification of mesh properties.

- **Example:**

  ```python
  # Get the first mesh's visibility
  mesh_visibility = nv.meshes[0].visible

  # Set the first mesh's visibility
  nv.meshes[0].visible = False
  ```

---

##### All Options

The `NiiVue` class inherits from `OptionsMixin`, which provides properties for various visualization options. These properties correspond to Niivue's configuration options.

Below is a list of available options with their default values:

- `text_height` (_float_): `0.06`
- `colorbar_height` (_float_): `0.05`
- `crosshair_width` (_int_): `1`
- `ruler_width` (_int_): `4`
- `show_3d_crosshair` (_bool_): `False`
- `back_color` (_tuple_): `(0, 0, 0, 1)`
- `crosshair_color` (_tuple_): `(1, 0, 0, 1)`
- `font_color` (_tuple_): `(0.5, 0.5, 0.5, 1)`
- `selection_box_color` (_tuple_): `(1, 1, 1, 0.5)`
- `clip_plane_color` (_tuple_): `(0.7, 0, 0.7, 0.5)`
- `ruler_color` (_tuple_): `(1, 0, 0, 0.8)`
- `colorbar_margin` (_float_): `0.05`
- `trust_cal_min_max` (_bool_): `True`
- `clip_plane_hot_key` (_str_): `'KeyC'`
- `view_mode_hot_key` (_str_): `'KeyV'`
- `double_touch_timeout` (_int_): `500`
- `long_touch_timeout` (_int_): `1000`
- `key_debounce_time` (_int_): `50`
- `is_nearest_interpolation` (_bool_): `False`
- `is_resize_canvas` (_bool_): `True`
- `is_atlas_outline` (_bool_): `False`
- `is_ruler` (_bool_): `False`
- `is_colorbar` (_bool_): `False`
- `is_orient_cube` (_bool_): `False`
- `multiplanar_pad_pixels` (_int_): `0`
- `multiplanar_force_render` (_bool_): `False`
- `is_radiological_convention` (_bool_): `False`
- `mesh_thickness_on_2d` (_float_): `float('inf')`
- `drag_mode` (_DragMode_): `DragMode.CONTRAST`
- `yoke_3d_to_2d_zoom` (_bool_): `False`
- `is_depth_pick_mesh` (_bool_): `False`
- `is_corner_orientation_text` (_bool_): `False`
- `sagittal_nose_left` (_bool_): `False`
- `is_slice_mm` (_bool_): `False`
- `is_v1_slice_shader` (_bool_): `False`
- `is_high_resolution_capable` (_bool_): `True`
- `log_level` (_str_): `'info'`
- `loading_text` (_str_): `'waiting for images...'`
- `is_force_mouse_click_to_voxel_centers` (_bool_): `False`
- `drag_and_drop_enabled` (_bool_): `True`
- `drawing_enabled` (_bool_): `False`
- `pen_value` (_int_): `1`
- `flood_fill_neighbors` (_int_): `6`
- `is_filled_pen` (_bool_): `False`
- `thumbnail` (_str_): `''`
- `max_draw_undo_bitmaps` (_int_): `8`
- `slice_type` (_SliceType_): `SliceType.MULTIPLANAR`
- `mesh_x_ray` (_float_): `0.0`
- `is_anti_alias` (_bool_, optional): `None`
- `limit_frames_4d` (_float_): `float('nan')`
- `is_additive_blend` (_bool_): `False`
- `show_legend` (_bool_): `True`
- `legend_background_color` (_tuple_): `(0.3, 0.3, 0.3, 0.5)`
- `legend_text_color` (_tuple_): `(1.0, 1.0, 1.0, 1.0)`
- `multiplanar_layout` (_MultiplanarType_): `MultiplanarType.AUTO`
- `render_overlay_blend` (_float_): `1.0`

These options can be set either during initialization or by setting the corresponding properties after creating the widget.

- **Example:**

  ```python
  # Set options during initialization
  nv = NiiVue(
      is_colorbar=True,
      back_color=(1, 1, 1, 1),
      slice_type=SliceType.RENDER,
  )

  # Set options after initialization
  nv.show_3d_crosshair = True
  nv.crosshair_color = (0, 1, 0, 1)
  ```

---

### Volume

Represents a single volume (e.g., a NIfTI image) to be visualized.

```python
from ipyniivue._widget import Volume
```

#### Attributes

- `path` (_pathlib.Path_ or _str_): The file system path or URL to the volume file. Supports NIfTI format.
- `opacity` (_float_): The opacity of the volume, ranging from `0.0` (transparent) to `1.0` (opaque).
- `colormap` (_str_): The name of the colormap to use for visualization.
- `colorbar_visible` (_bool_): Whether to display the colorbar associated with the volume.
- `cal_min` (_float_, optional): The minimum intensity value for calibration. If `None`, uses the volume's minimum value.
- `cal_max` (_float_, optional): The maximum intensity value for calibration. If `None`, uses the volume's maximum value.

- **Example:**

  ```python
  volume = Volume(
      path='path/to/volume.nii',
      opacity=0.8,
      colormap='gray',
      colorbar_visible=True,
      cal_min=0.0,
      cal_max=1.0,
  )
  ```

---

### Mesh

Represents a 3D mesh to be visualized.

```python
from ipyniivue._widget import Mesh
```

#### Attributes

- `path` (_pathlib.Path_ or _str_): The file system path or URL to the mesh file.
- `rgba255` (_list_ of 4 ints): The RGBA color of the mesh, with each value in the range `[0, 255]`.
- `opacity` (_float_): The opacity of the mesh, ranging from `0.0` (transparent) to `1.0` (opaque).
- `visible` (_bool_): Whether the mesh is visible.
- `layers` (_list_): A list of additional mesh layers.

- **Example:**

  ```python
  mesh = Mesh(
      path='path/to/mesh.obj',
      rgba255=[255, 0, 0, 255],
      opacity=0.5,
      visible=True,
      layers=[],
  )
  ```

---

### WidgetObserver

A utility class to observe and synchronize property changes between widgets and objects.

#### Initialization

```python
WidgetObserver(widget, object, attribute)
```

- **Parameters:**
  - `widget`: The widget to observe (e.g., an ipywidgets control).
  - `object`: The object whose attribute should be updated when the widget changes.
  - `attribute` (_str_): The name of the attribute on the `object` to update.

- **Description:** Sets up an observer so that when the `widget` changes, the `attribute` on the `object` is updated accordingly.

- **Example:**

  ```python
  import ipywidgets as widgets
  from ipyniivue import NiiVue, WidgetObserver

  nv = NiiVue()
  slider = widgets.FloatSlider(min=0, max=1, step=0.1, value=1.0)
  widget_observer = WidgetObserver(widget=slider, object=nv.volumes[0], attribute='opacity')
  ```

---

## Examples

### Basic Usage

```python
from ipyniivue import NiiVue, SliceType

# Initialize a NiiVue widget
nv = NiiVue(slice_type=SliceType.MULTIPLANAR)

# Load volumes
nv.load_volumes([
    {
        'path': 'path/to/brain.nii',
        'colormap': 'gray',
        'opacity': 1.0,
    },
    {
        'path': 'path/to/overlay.nii',
        'colormap': 'hot',
        'opacity': 0.5,
    },
])

# Display the widget
nv
```

### Modifying Volume Properties

```python
# Set the opacity of the first volume
nv.volumes[0].opacity = 0.8

# Change the colormap of the second volume
nv.volumes[1].colormap = 'cool'
```

### Using WidgetObserver

```python
import ipywidgets as widgets
from ipyniivue import NiiVue, WidgetObserver

nv = NiiVue()

# Create a slider to control opacity
opacity_slider = widgets.FloatSlider(min=0, max=1, step=0.1, value=1.0, description='Opacity')

# Set up the observer
widget_observer = WidgetObserver(widget=opacity_slider, object=nv.volumes[0], attribute='opacity')

# Display the widget and the slider
display(nv, opacity_slider)
```

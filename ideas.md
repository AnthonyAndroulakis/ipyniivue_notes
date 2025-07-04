# on-event
how to handle loading images on frontend -> update to backend
- cannot use built in on-events, because:
```ts
  async addMeshFromUrl(meshOptions: LoadFromUrlParams): Promise<NVMesh> {
    const ext = this.getFileExt(meshOptions.url)
    if (ext === 'JCON' || ext === 'JSON') {
      const response = await fetch(meshOptions.url, {})
      const json = await response.json()
      const mesh = this.loadConnectomeAsMesh(json)
      this.mediaUrlMap.set(mesh, meshOptions.url)
      this.onMeshAddedFromUrl(meshOptions, mesh)
      this.addMesh(mesh)
      return mesh
    }
    const mesh = await NVMesh.loadFromUrl({ ...meshOptions, gl: this.gl })
    this.mediaUrlMap.set(mesh, meshOptions.url)
    this.onMeshAddedFromUrl(meshOptions, mesh)
    this.addMesh(mesh)
    return mesh
  }
```
here the onMeshAddedFromUrl gets called before addMesh, so on mesh_added_from_url won't be able to point to the loaded mesh properly... it'll just be...a mesh that hasnt been added to nv...

was originally thinking to have the mesh added the moment niivue access to it...but that doesnt make sense here since addMesh gets called afterwards. So, there're really 2 solutions:
1. don't find the loaded mesh when it doesnt exist
2. change niivue (not an option...)

there're 2 issues to be solved in this specific problem:
1. on_mesh_added_from_url mesh object
2. on_mesh_loaded mesh object



could also...just have the mesh data going to the callback function just be the volume/mesh id...and let the end user deal with it...but thats not very user friendly nor pythonic

# volumes loaded from frontend
volume loaded in frontend -> send "add_volume" event to backend with "preloaded" path -> backend receives and adds volume to _volumes -> frontend receives update due to _volumes change -> frontend verifies that backend and frontend volumes are the same (based on ids)

same occurs for meshes

1 slight problem: what if a user inputs a volume that isn't valid. Say, a user tries to add a volume manually that contains the following data:
```json
{
    "path": "preloaded"
    "id": ""
}
```
this would break ipyniivue since this is an invalid volume that passes the checks for file_serializer:
```py
def file_serializer(instance: typing.Union[pathlib.Path, str], widget: object):
    if isinstance(instance, str):
        if instance == "preloaded":
            return {"name": "preloaded", "data": ""}
        # make sure we have a pathlib.Path instance
        instance = pathlib.Path(instance)
    return {"name": instance.name, "data": instance.read_bytes()}
```

We could verify that the volume has a valid v4 uuid, but the user can generate a fake v4 uuid.

An easy simple solution is to simply filter out these invalid volumes in add_volume and load_volumes, and filter out invalid meshes in add_mesh and load_meshes, but that still leaves the issue of directly modifiying nv._volumes. 

One could then further restrict modification of nv._volumes by changing it to nv.__volumes and then modifying setattr:
```py
def __setattr__(self, name, value):
    if name in {'_NiiVue__volumes', '_NiiVue__meshes'}:
        raise AttributeError("Direct modification of volumes or meshes is not allowed.")
    super().__setattr__(name, value)
```
but thats...too messy. And it wouldnt work completely, cause then what if someone just calls nv._add_volume_from_frontend(...) or sends a fake WebSocket event that tells the backend to add a fake volume.

What would work completely, is to verify that volumes added on the backend are valid.

Although...technically...a user could just add a duplicate id to the volumes array and break stuff that way...so...whats the solution then?

# loadvolumefromurl + loadmeshfromurl
2 ways to implement:
1. send msg with url(s) to frontend and have frontend load in the data -> update backend (might not be very good for future function implementations)
2. have path support urls (for serialization...it just converts to data every time? is this really a good idea?)

3. or...3rd option, change the format of volumes + meshes so that it contains path + data, and so path doesnt get serialized as {name:..., data:...} but instead path is just path string and data gets set on initialization. (I think this + a combination of idea 2 is best).

# funny case of timing: setCrosshairWidth implementation
```ts
  setCrosshairWidth(crosshairWidth: number): void {
    this.opts.crosshairWidth = crosshairWidth
    if (this.crosshairs3D) {
      this.crosshairs3D.mm![0] = NaN // force redraw
    }
    this.drawScene()
  }
```
doing self.crosshair_width = <num> will...
1. cause a change in self._opts
2. which will propagate into the frontend and cause:
```ts
  nv.document.opts = { ...nv.opts, ...model.get("_opts") };
  nv.drawScene();
  nv.updateGLVolume();
```

so, we can't just do self.crosshair_width = <num> since that won't force redraw of the crosshair. Perhaps we can send a msg to the frontend telling it to set crosshairs3D.mm to NaN, and then do self.crosshair_width = <nan> to update the width and redraw, which should work...but will the timing be preserved? Will draw happen after the frontend crosshairs3D has occurred?

other option is to somehow change the opts -> have the propagate into the frontend -> but not drawscene or update volume....no..thats messy

or...3rd option, we set crosshair width (it does whatever it wants to)
we also send a msg, for it to set the mm to NaN AND draw the scene....

oh wait, there's a 4th option lol...we modify the change callback in ts, thats clean yay
```ts
		model.on("change:_opts", () => {
			nv.document.opts = { ...nv.opts, ...model.get("_opts") };
			nv.drawScene();
			nv.updateGLVolume();
		});
```
just add some if statement to catch opt === crosshairWidth, yay, and that'll be helpful for other setters that need more involved answers...

well, it's not *that* simple, but need to check to see if certain changes exist in model.get("opts"), using switch case statements with omission of break statements in certain spots



# Rerendering
```js
async function forceReRender(
	nv: niivue.Niivue,
	model: Model,
	disposer: Disposer,
) {
	let canvas: HTMLCanvasElement;
	let container: HTMLElement;

	// Check and remove existing canvas
	if (nv.canvas?.parentNode) {
		container = nv.canvas.parentNode as HTMLElement;

		container.removeChild(nv.canvas);

		// Create a new canvas
		canvas = document.createElement("canvas");
		container.appendChild(canvas);

		// Height changes
		model.off("change:height");
		model.on("change:height", () => {
			container.style.height = `${model.get("height")}px`;
		});

		// Attach nv to the new canvas
		nv.attachToCanvas(canvas, nv.opts.isAntiAlias);

		// Update gl volume
		nv.updateGLVolume();
	}
}
```
dunno..

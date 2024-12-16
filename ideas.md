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

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

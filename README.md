# true3d-format-spec
A specification for reprsenting true 3D data.

## Directory Layout

Data stored in true3d format have following on-disk layout:

```
root/manifest.json
    /window0/0/
              /mesh.obj
              /mesh.mtl
              /texture0.jp2
              /texture1.jp2
              ...
           /1/
              ...
           ...
            
```

The root directory contains a `manifest.json` file with export metadata (described later in this document). 3D data are divided into ''windows'' -- generalized 3D tiles. Each window is made of multiple version of the same data: original and derived levels of detail (LODs). The ratio of average texel size (mesh area in meters divided by texture area in pixels) between LOD and LOD+1 is expected to be 2. Physical data are textured 3D meshes stored as a .OBJ file (optinally gzipped) with atlas: list of textures and helper MTL file for simple meshlab display. All meshes share the same origin (0, 0, 0).

Optionally, whole directory structure can be archived in a tar archive. Format reader should be able to locate data by finding `manifest.json` file.

## Manifest

A manifest file contains all export metadata at one place. Manifest is written as a regular JSON file.

All paths in manifest are relative to path of enclosing entity: `window` path is relative to `manifest.json` path, `lod` path is relative to the `window` it belongs etc, atlas textures and mesh paths are relative to their `lod`.

### Manifest example:
```javascript
{
    // format version; currently; currently, only version 1 is supported
    "version": 1

    // spatial reference system used to transfrom from XYZ coordinates to geo space
    // missing if this is purely local XYZ
    // available formats: Proj.4 format, WKT string, EPSG:code or ENU
    // ENU format is described in separate document
    , "srs": "+proj=utm +zone=33 +datum=WGS84 +no_defs"
 
    , "windows": [
        {
            // path to window content (relative to this file)
            "path": "window0"

            // levels of detail, sorted from most detailed verion (original) to the coarsest one
            "lods": [
                {
                    // path to LOD data (relative to window path)
                    "path": "0"

                    // atlas: a list of textures referenced by mesh
                    , "atlas": [
                        {
                            // path to texture file, relative to window/lod path
                            "path": "texture0.jpg"
                            // texture size in pixels
                            , "size": [ 1024, 512 ]
                            // texture format (one of jpg, png, jpeg2000)
                            , "format": "jpg"
                        }
                        , ...
                    ]

                    // mesh
                    , "mesh": {
                        // path to mesh, relative to window/lod path
                        "path": "mesh.obj"
                        // mesh format (one of obj, obj.gz)
                        "format": "obj"
                    }
                }
            ]
        }
        , ...
    ]
}
```

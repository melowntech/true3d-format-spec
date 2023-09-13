# VEF format specification
A specification for representing hierarchical true 3D data.

# License

The text of this specification is licensed under a [Creative Commons Attribution 3.0 United States License](http://creativecommons.org/licenses/by/3.0/us/). However, the use of this specification in products and code is entirely free. There are no royalties, restrictions, or requirements.

## Directory Layout

Data stored in true3d format have following on-disk layout:

```
root/manifest.json
    /window0/0/
              /mesh.obj
              /mesh.mtl
              /texture0.jpg
              /texture1.jpg
              ...
            /1/
              ...
           ...
    /window1/0/
           ...
```

The root directory contains a `manifest.json` file with export metadata (described later in this document). 3D data are divided into ''windows'' -- generalized 3D tiles. Each window is made of multiple version of the same data: original and derived levels of detail (LODs). The ratio of average texel size (mesh area in meters divided by texture area in pixels) between LOD and LOD+1 is expected to be 2. Highest number (LOD) corresponds to the finest mesh. Physical data are textured 3D meshes stored as a .OBJ file (optinally gzipped) with atlas: list of textures and helper MTL file for simple meshlab display.

Texture assignment to faces is done via `usemtl IDX` lines in OBJ files where IDX is numeric index of a texture in the atlas, i.e. faces from first texture follow `usemtl 0` line, faces from second texture follow `usemtl 1` line etc.

If part of the mesh is untextured then the atlas entry may be either empty or contain a RGB color reference. Provided color can be used as a material where appropripate. If atlas is smaller than number of submeshes in the actual mesh the implementation should treat material references outside of the valid atlas as faces belonging to an untextured sub-meshes.

Optionally, whole directory structure can be archived in a TAR or a ZIP archive. Format reader should be able to locate data by finding `manifest.json` file inside the archive. TAR should not be compressed to allow dirrect file access. ZIP might use compression since ZIP uses per-file compression.

Directory structure inside format is not mandated since all paths are recorded inside the manifest.

All 3D meshes share same origin (i.e. point 0, 0, 0). Meshes are georeferenced by using spatial reference system.

Since coordinates in most SRS are huge numbers vertices can be transformed to much smaller number by applying a transformation. For example, vertices can be localized around common center. Inverse transformation matrix **A** for data can be stored at the manifest level (applies to all meshes) or at window level (applies to all meshes in given window). If **A** is defined, every point *x* of given mesh is first expressed in homogenous coordinates and then transformed as **A** *x* before further processing.

## Manifest

A manifest file contains all export metadata at one place. Manifest is written as a regular JSON file.

All paths in manifest are relative to path of enclosing entity: `window` path is relative to `manifest.json` path, `lod` path is relative to the `window` it belongs etc, atlas textures and mesh paths are relative to their `lod`.

### Transformation matrix

Transformation (3x4 row major) matrices are represented as 12-element array, i.e. matrix:
```
    / a b c d \
A = | e f g h |
    \ i j k l /
```
is stored as
```javascript
[ a, b, c, d, e, f, g, h, i, j, k, l ]
```

### 3D Extents

3D mesh extents (axis-aligned bounding box) written in the format `[ min.x, mib.y, min,z, max.x, max.y, max.z ]`.

### SRS

A spatial reference system can be written in one of the following formats:

 * +proj format string
 * WKT string
 * EPSG code in format EPSG:number, e.e. EPSG:3857 for pseudomercator
 * combined EPSG horizontal and vertical system codes in format EPSG:number+number, e.g. EPSG:3857+5773 for pseudomercator combined with EGM96-based vertical system
 * custom ENU (east, north, up) format for local cartesian system at given place on the earth surface
 
See [ENU format specification](enu.md) for more information.

### Manifest example:
```javascript
{
    // format version; currently; currently, only version 1 is supported
    "version": 1

    // spatial reference system used to transfrom from XYZ coordinates to geo space
    // missing if this is purely local XYZ
    // available formats: Proj.4 format, WKT string, EPSG:code, EPSG:code+code or ENU
    // ENU format is described in separate document
    , "srs": "+proj=utm +zone=33 +datum=WGS84 +no_defs"
    
    // optional transformation to be applied to all meshes
    , "trafo": [ 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0 ]

    // optional Ground Sampling Distanc [in meters] of finest detail mesh
    // may be overridden at window level
    , "gsd": 0.05 // 5 cm
    
    , "windows": [
        {
            // path to window content (relative to this file)
            "path": "window0"
 
            // optional transformation to be applied to all meshes in this window
            , "trafo": [ 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0 ]
            
            // optional 3D extents (axia-aligned bounding box) of window mesh, in global coordinate system;
            // used for optimization purposes
            // example: bbox with extremes at [0, 0, 0] and [100, 100, 100]:
            , "extents": [ 0, 0, 0, 100, 100, 100 ]
 
            // levels of detail, sorted from most detailed verion (original) to the coarsest one
            , "lods": [
                {
                    // path to LOD data (relative to window path)
                    "path": "0"

                    // optional Ground Sampling Distanc [in meters] of finest detail mesh
                    // overrides top-level GSD
                    // if there's no GSD available for window, implementation must calculate GSD from mesh and atlas data
                    "gsd": 0.04 // 4.5 cm, overrides 5 cm archive-wide default

                    // atlas: A list of textures referenced by mesh
                    , "atlas": [
                        {
                            // textured sub-mesh

                            // path to texture file, relative to window/lod path
                            "path": "texture0.jpg"
                            // texture size in pixels
                            , "size": [ 1024, 512 ]
                            // texture format (one of jpg, png, jpeg2000)
                            , "format": "jpg"
                        }
                        , {
                            // untextured sub-mesh, GSD is mandatory
                            // material properties unspecified
                        }
                        , {
                            // untextured sub-mesh, GSD is mandatory
                            // material color
                            "color": [ 255, 255, 0 ] // yellow color
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


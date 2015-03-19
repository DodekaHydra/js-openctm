**js-openctm** is a JavaScript library for reading OpenCTM files.

<a href='http://openctm.sourceforge.net'>OpenCTM</a> is a file format for storing 3D triangle meshes created by Marcus Geelnard.

Some parts of this library are a direct traslation to JavaScript from original C source code found on the OpenCTM SDK.



# Demo #

- <a href='http://www.inmensia.com/files/ctm/demo/demo.html'>Live demo!</a>

- <a href='http://alteredqualia.com/three/examples/webgl_loader_ctm_materials.html'>Using Three.js to render</a>

# Video #

<a href='http://www.youtube.com/watch?feature=player_embedded&v=mCjs9oWlccg' target='_blank'><img src='http://img.youtube.com/vi/mCjs9oWlccg/0.jpg' width='425' height=344 /></a>

# How to use it? #

**1)** Retrieve your `.ctm` file from the web, as usual:

```
function retrieve(url){
  var request = new XMLHttpRequest();
  request.open("GET", url, false);
  request.overrideMimeType("text/plain; charset=x-user-defined");
  request.send();
  if ( (200 !== request.status) && (0 !== request.status) ){
    throw new Error(request.status + " Retrieving " + url); 
  }
  return request;
};

var request = retrieve("<your .ctm filename url here>");
```

**2)** Create a `CTM.Stream` object from the retrieved file:

```
var stream = new CTM.Stream(request.responseText);
```

**3)** Create a `CTM.File` object from the stream:

```
var file = new CTM.File(stream);
```

# What you get? #

`CTM.File` objects have two properties:

  * `header`, a object with the following properties:
    * `fileFormat`: File format version (always 5)
    * `compressionMethod`: Compression method (RAW, MG1 or MG2)
    * `vertexCount`: Vertex count
    * `triangleCount`: Triangle count
    * `uvMapCount`: UV map count
    * `attrMapCount`: Attribute map count
    * `flags`: Boolean flags (bit 0 means that normals are present)
    * `comment`: File comment

  * `body`, a object with the following properties:
    * `indices`: A `Uint32Array` buffer with 3 x `header.triangleCount` elements
    * `vertices`: A `Float32Array` buffer with `header.vertexCount` elements
    * `normals`: (optional) A `Float32Array` buffer with `header.vertexCount` elements
    * `uvMaps`: (optional) An array with `header.uvMapCount` elements, each of them with the following properties:
      * `name`: UV map name
      * `filename`: UV map ﬁle name
      * `uv`: A `Float32Array` buffer with 2 x `header.vertexCount` elements
    * `attrMaps`: (optional) A array with `header.attrMapCount` elements, each of them with the following properties:
      * `name`: Attribute map name
      * `attr`: A `Float32Array` buffer with 4 x `header.vertexCount` elements

# How to draw the mesh? #

Using WebGL, of course.

```
...
indicesBuffer = gl.createBuffer();
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indicesBuffer);
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(file.body.indices), gl.STATIC_DRAW);

verticesBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, verticesBuffer);
gl.bufferData(gl.ARRAY_BUFFER, file.body.vertices, gl.STATIC_DRAW);

if (file.body.normals){
  normalsBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, normalsBuffer);
  gl.bufferData(gl.ARRAY_BUFFER,file.body.normals, gl.STATIC_DRAW);
}

if (file.body.uvMaps){
  uvBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, uvBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, file.body.uvMaps[0].uv, gl.STATIC_DRAW);
}
...
```

No idea? Try <a href='http://learningwebgl.com/blog/?page_id=1217'><a href='http://learningwebgl.com'>http://learningwebgl.com</a><a />.<br>
<br>
<h1>Dependencies</h1>
<a href='http://js-lzma.googlecode.com'>js-lzma</a>: A port of LZMA decompression algorithm to JavaScript.<br>
<br>
<h1>Limitations</h1>

- Textures are not packed on <code>.ctm</code> files, you must retrieve them on your own.<br>
<br>
- Indices are exposed like a <code>Uint32Array</code> buffer, but please, remember that WebGL <code>drawElements</code> function works with <code>Uint16Array</code>, you must split them on your own.<br>
<br>
<h1>What about writer function?</h1>

Sorry, don't. This library is part of another project of mine that only needed the reader algorithm.
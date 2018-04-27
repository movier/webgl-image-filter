<template>
  <canvas id="glCanvas" width="553" height="415" />
</template>

<script>


//
// Initialize a shader program, so WebGL knows how to draw our data
//
function initShaderProgram(gl, vsSource, fsSource) {
  const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
  const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);

  // Create the shader program

  const shaderProgram = gl.createProgram();
  gl.attachShader(shaderProgram, vertexShader);
  gl.attachShader(shaderProgram, fragmentShader);
  gl.linkProgram(shaderProgram);

  // If creating the shader program failed, alert

  if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
    alert('Unable to initialize the shader program: ' + gl.getProgramInfoLog(shaderProgram));
    return null;
  }

  return shaderProgram;
}

//
// creates a shader of the given type, uploads the source and
// compiles it.
//
function loadShader(gl, type, source) {
  const shader = gl.createShader(type);

  // Send the source to the shader object

  gl.shaderSource(shader, source);

  // Compile the shader program

  gl.compileShader(shader);

  // See if it compiled successfully

  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    alert('An error occurred compiling the shaders: ' + gl.getShaderInfoLog(shader));
    gl.deleteShader(shader);
    return null;
  }

  return shader;
}

function initBuffers(gl) {

  // Create a buffer for the square's positions.

  const positionBuffer = gl.createBuffer();

  // Select the positionBuffer as the one to apply buffer
  // operations to from here out.

  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);

  // Now create an array of positions for the square.

  const positions = [
    -1, -1, -1, 1, 1, -1,
  1, -1, 1, 1, -1, 1, 
  ]

  // Now pass the list of positions into WebGL to build the
  // shape. We do this by creating a Float32Array from the
  // JavaScript array, then use it to fill the current buffer.

  // gl.STATIC_DRAW tells WebGL that the data are not likely to change.
  gl.bufferData(gl.ARRAY_BUFFER,
                new Float32Array(positions),
                gl.STATIC_DRAW);
}

function draw() {
  const canvas = document.querySelector("#glCanvas");
  // Initialize the GL context
  const gl = canvas.getContext("webgl");
  // Only continue if WebGL is available and working
  if (!gl) {
    alert("Unable to initialize WebGL. Your browser or machine may not support it.");
    return;
  }

  gl.clearColor(1, 1, 1, 1);
  gl.clear(gl.COLOR_BUFFER_BIT);


  initBuffers(gl);

  // Vertex shader program
  const vsSource = `
    attribute vec2 position;
    varying vec2 v_coord;

    void main() {
      gl_Position = vec4(position, 0, 1);
      v_coord = gl_Position.xy * 0.5 + 0.5;
    }
  `;

  const fsSource = `
    precision mediump float;
    varying vec2 v_coord;
    uniform sampler2D u_texture;
    vec4 newColor;

    void main() {
      vec4 sampleColor = texture2D(u_texture, vec2(v_coord.x, 1.0 - v_coord.y));
      newColor = sampleColor;
      if (v_coord.x < 0.5) {
        newColor.r = (sampleColor.r * .393) + (sampleColor.g * .769) + (sampleColor.b * .189);
        newColor.g = (sampleColor.r * .349) + (sampleColor.g * .686) + (sampleColor.b * .168);
        newColor.b = (sampleColor.r * .272) + (sampleColor.g * .534) + (sampleColor.b * .131);
      }
      gl_FragColor = newColor;
    }
  `;

  const program = initShaderProgram(gl, vsSource, fsSource);
  const positionAttributeLocation = gl.getAttribLocation(program, 'position');

  gl.useProgram(program);
  gl.enableVertexAttribArray(positionAttributeLocation);

  // gl.vertexAttribPointer(location, size, type, normalize, stride, offset)
  // This method bind ARRAY_BUFFER to specified attribute
  gl.vertexAttribPointer(positionAttributeLocation, 2, gl.FLOAT, false, 0, 0);

  const texture = gl.createTexture();
  texture.image = new Image();
  texture.image.onload = function () {
    gl.bindTexture(gl.TEXTURE_2D, texture);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, texture.image);
    gl.drawArrays(gl.TRIANGLES, 0, 6);
  };
  texture.image.crossOrigin = '';
  // texture.image.src = 'http://davidguan.me/webgl-intro/filter/github.jpg';
  texture.image.src = 'https://farm5.staticflickr.com/4485/24263717138_bde9950b28_h.jpg';
}

export default {
  name: 'HelloWorld',
  mounted() {
    draw();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

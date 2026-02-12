<script>
  import { onMount } from "svelte";

  let maxClicks = 10;
  let usedClicks = 0;
  $: remaining = maxClicks - usedClicks;

  let canvas;
  let gl;
  let program;
  let startTime = Date.now();

  function increase() {
    if (usedClicks < maxClicks) {
      usedClicks++;
    }
  }

  function decrease() {
    if (usedClicks > 0) {
      usedClicks--;
    }
  }

  onMount(() => {
    gl = canvas.getContext("webgl");

    const vertexShaderSource = `
      attribute vec2 position;
      varying vec2 vUv;
      void main() {
        vUv = position * 0.5 + 0.5;
        gl_Position = vec4(position, 0.0, 1.0);
      }
    `;

    const fragmentShaderSource = `
      precision mediump float;
      varying vec2 vUv;
      uniform float u_time;
      uniform float u_strength;

      void main() {
        float wave1 = sin(vUv.x * 10.0 + u_time * 2.0);
        float wave2 = sin(vUv.y * 8.0 - u_time * 1.5);
        float wave = (wave1 + wave2) * 0.5;

        float intensity = wave * u_strength;

        vec3 deepBlue = vec3(0.0, 0.2, 0.5);
        vec3 lightBlue = vec3(0.2, 0.6, 1.0);

        float gradient = vUv.y;
        vec3 color = mix(deepBlue, lightBlue, gradient);

        float highlight = smoothstep(0.4, 0.8, intensity + 0.5);

        gl_FragColor = vec4(color + highlight * 0.3, 1.0);
      }
    `;

    function compile(type, source) {
      const shader = gl.createShader(type);
      gl.shaderSource(shader, source);
      gl.compileShader(shader);
      return shader;
    }

    const vertexShader = compile(gl.VERTEX_SHADER, vertexShaderSource);
    const fragmentShader = compile(gl.FRAGMENT_SHADER, fragmentShaderSource);

    program = gl.createProgram();
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);
    gl.useProgram(program);

    const buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
        -1, -1,
         1, -1,
        -1,  1,
         1,  1
      ]),
      gl.STATIC_DRAW
    );

    const position = gl.getAttribLocation(program, "position");
    gl.enableVertexAttribArray(position);
    gl.vertexAttribPointer(position, 2, gl.FLOAT, false, 0, 0);

    function render() {
      const time = (Date.now() - startTime) / 1000;

      gl.uniform1f(
        gl.getUniformLocation(program, "u_time"),
        time
      );

      // 关键：点击越多，波浪越强
      const strength = usedClicks / maxClicks;
      gl.uniform1f(
        gl.getUniformLocation(program, "u_strength"),
        strength
      );

      gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
      requestAnimationFrame(render);
    }

    render();
  });
</script>

<h1>Interactive Water Visualization</h1>

<div class="controls">
  <p>Remaining Clicks: {remaining}</p>

  <button on:click={increase}>Increase</button>
  <button on:click={decrease}>Decrease</button>

  <br /><br />

  <label>
    Set Max Clicks:
    <input type="number" bind:value={maxClicks} min="1" />
  </label>
</div>

<canvas bind:this={canvas} width="800" height="400"></canvas>

<style>
  h1 {
    font-size: 28px;
  }

  .controls {
    margin-bottom: 20px;
  }

  button {
    background-color: #3498db;
    font-size: large;
    margin-right: 10px;
    padding: 10px;
    border: none;
    color: white;
    cursor: pointer;
  }

  button:hover {
    background-color: #2980b9;
  }

  canvas {
    border: 2px solid #ccc;
    display: block;
  }
</style>

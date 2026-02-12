<script>
    import { onMount } from "svelte";

    let canvas;

    onMount(() => {
        const gl = canvas.getContext("webgl");

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
            uniform float uTime;

            float wave(vec2 uv) {
                float w = sin(uv.x * 20.0 + uTime * 2.0) * 0.08;
                w += cos(uv.y * 15.0 + uTime * 1.5) * 0.06;
                w += sin((uv.x + uv.y) * 10.0 + uTime) * 0.05;
                return w;
            }

            void main() {
                float h = wave(vUv);

                vec3 deepWater = vec3(0.0, 0.2, 0.5);
                vec3 shallowWater = vec3(0.0, 0.5, 0.9);

                float fresnel = pow(1.0 - vUv.y, 3.0);

                vec3 color = mix(deepWater, shallowWater, h + 0.5);

                float highlight = pow(abs(h), 2.0) * 5.0;

                vec3 finalColor = color + highlight + fresnel * 0.3;

                gl_FragColor = vec4(finalColor, 1.0);
            }
        `;

        function createShader(type, source) {
            const shader = gl.createShader(type);
            gl.shaderSource(shader, source);
            gl.compileShader(shader);
            return shader;
        }

        const vertexShader = createShader(gl.VERTEX_SHADER, vertexShaderSource);
        const fragmentShader = createShader(gl.FRAGMENT_SHADER, fragmentShaderSource);

        const program = gl.createProgram();
        gl.attachShader(program, vertexShader);
        gl.attachShader(program, fragmentShader);
        gl.linkProgram(program);
        gl.useProgram(program);

        const positionLocation = gl.getAttribLocation(program, "position");
        const timeLocation = gl.getUniformLocation(program, "uTime");

        const buffer = gl.createBuffer();
        gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
        gl.bufferData(
            gl.ARRAY_BUFFER,
            new Float32Array([
                -1, -1,
                 1, -1,
                -1,  1,
                 1,  1,
            ]),
            gl.STATIC_DRAW
        );

        gl.enableVertexAttribArray(positionLocation);
        gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);

        function render(time) {
            gl.viewport(0, 0, canvas.width, canvas.height);
            gl.clearColor(0.0, 0.0, 0.0, 1.0);
            gl.clear(gl.COLOR_BUFFER_BIT);

            gl.uniform1f(timeLocation, time * 0.001);

            gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
            requestAnimationFrame(render);
        }

        requestAnimationFrame(render);
    });
</script>

<canvas bind:this={canvas} width="900" height="600"></canvas>

<style>
    canvas {
        display: block;
        margin: 0 auto;
        border: 1px solid #ccc;
    }
</style>

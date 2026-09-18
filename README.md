# Proyecto-Integrador
Proyecto Integrador - Calculadora básica
<!DOCTYPE html>
<html lang="es">

<head>

    <!-- Permite usar caracteres como á, é, í, ó y ú -->
    <meta charset="utf-8">

    <!-- Hace que la página se adapte a celulares y computadores -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Título que aparece en la pestaña del navegador -->
    <title>Calculadora de Volúmenes</title>

    <!-- Librería Tailwind CSS para darle estilos a la página -->
    <script src="https://cdn.tailwindcss.com?plugins=forms,container-queries"></script>

    <!-- Librería FontAwesome para utilizar iconos -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">


    <style>

        /* Importamos las fuentes que vamos a utilizar */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400&display=swap');


        /* Estilos generales del cuerpo de la página */
        body {

            /* Tipo de letra */
            font-family: 'Inter', sans-serif;

            /* Fondo negro */
            background-color: #000000;

            /* Texto blanco */
            color: #ffffff;

            /* Evita que aparezca desplazamiento horizontal */
            overflow-x: hidden;
        }


        /* Clase para utilizar una fuente tipo código */
        .font-mono {

            font-family: 'JetBrains Mono', monospace;
        }


        /* Sombra personalizada */
        .shadow-custom {

            box-shadow:
                0 4px 6px -1px rgba(0, 0, 0, 0.5),
                0 2px 4px -1px rgba(0, 0, 0, 0.3);
        }


        /* Sombra interna */
        .shadow-inner-custom {

            box-shadow:
                inset 0 2px 4px 0 rgba(0, 0, 0, 0.3);
        }


        /* Estilo de la pestaña que está seleccionada */
        .tab-active {

            /* Borde verde */
            border: 1px solid #00FF00;

            /* Texto blanco */
            color: #ffffff;

            /* Fondo con degradado */
            background: linear-gradient(
                135deg,
                rgba(0, 255, 0, 0.15),
                rgba(0, 123, 255, 0.15)
            );

            /* Sombra verde */
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.2);

            /* Efecto de desenfoque */
            backdrop-filter: blur(12px);
        }


        /* Estilo de las pestañas que no están seleccionadas */
        .tab-inactive {

            /* Borde transparente */
            border: 1px solid rgba(255, 255, 255, 0.1);

            /* Texto gris */
            color: #a1a1aa;

            /* Fondo oscuro */
            background-color: rgba(0, 0, 0, 0.4);

            /* Desenfoque */
            backdrop-filter: blur(12px);
        }


        /* Estilo del botón activo de la navegación inferior */
        .nav-active {

            /* Fondo degradado */
            background: linear-gradient(
                135deg,
                rgba(0, 255, 0, 0.2),
                rgba(0, 123, 255, 0.2)
            );

            /* Color verde */
            color: #00FF00;

            /* Bordes redondos */
            border-radius: 9999px;

            /* Sombra */
            box-shadow: 0 0 15px rgba(0, 255, 0, 0.15);
        }


        /* Estilo del área donde aparece el resultado */
        .glow-result {

            /* Sombra azul y verde */
            box-shadow:
                0 0 25px rgba(0, 123, 255, 0.3),
                inset 0 0 20px rgba(0, 255, 0, 0.1);

            /* Borde azul */
            border: 1px solid #007BFF;
        }


        /* Estilo del botón de calcular */
        .btn-gradient {

            /* Fondo degradado */
            background: linear-gradient(
                135deg,
                #007BFF,
                #00FF00
            );

            /* Texto negro */
            color: black;
        }


        /* Cuando pasamos el mouse por encima del botón */
        .btn-gradient:hover {

            /* Cambia el degradado */
            background: linear-gradient(
                135deg,
                #0056b3,
                #00cc00
            );
        }


        /* Esta clase sirve para ocultar elementos */
        .hidden-input {

            display: none;
        }

    </style>

</head>


<body class="flex flex-col min-h-screen pb-20 relative">


    <!-- ================================================= -->
    <!-- FONDO ANIMADO -->
    <!-- ================================================= -->

    <!-- Canvas donde se dibuja el fondo animado -->
    <canvas
        class="fixed inset-0 w-full h-full -z-10"
        id="shader-canvas-ANIMATION_7">
    </canvas>


    <script>

        // Función que contiene todo el código del fondo
        (function() {

            // Buscamos el canvas por su ID
            const canvas =
                document.getElementById('shader-canvas-ANIMATION_7');


            // Función para ajustar el tamaño del canvas
            function syncSize() {

                // Obtenemos el ancho
                const w = canvas.clientWidth || 1280;

                // Obtenemos el alto
                const h = canvas.clientHeight || 720;


                // Comprobamos si el tamaño cambió
                if (canvas.width !== w || canvas.height !== h) {

                    // Cambiamos el ancho
                    canvas.width = w;

                    // Cambiamos el alto
                    canvas.height = h;
                }
            }


            // Detectamos cuando cambia el tamaño de la ventana
            if (typeof ResizeObserver !== 'undefined') {

                new ResizeObserver(syncSize).observe(canvas);
            }


            // Ajustamos el tamaño inicialmente
            syncSize();


            // Creamos el contexto WebGL
            const gl =
                canvas.getContext('webgl') ||
                canvas.getContext('experimental-webgl');


            // Si el navegador no soporta WebGL, terminamos
            if (!gl) return;


            // Código del vertex shader
            const vs = `
                attribute vec2 a_position;

                varying vec2 v_texCoord;

                void main() {

                    v_texCoord = a_position * 0.5 + 0.5;

                    gl_Position =
                        vec4(a_position, 0.0, 1.0);
                }
            `;


            // Código del fragment shader
            const fs = `
                precision highp float;

                varying vec2 v_texCoord;

                uniform float u_time;
                uniform vec2 u_resolution;
                uniform vec2 u_mouse;
                uniform float u_scale;


                void main() {

                    // Obtenemos la posición del pixel
                    vec2 uv = v_texCoord;


                    // Utilizamos el tiempo para crear movimiento
                    float time = u_time * 0.5;


                    // Color base oscuro
                    vec3 color =
                        vec3(0.005, 0.005, 0.01);


                    // Creamos un punto central
                    vec2 center = vec2(0.5);


                    // Calculamos la distancia al centro
                    float dist =
                        length(uv - center);


                    // Creamos un efecto de pulsación
                    float pulse =
                        0.5 + 0.2 * sin(time * 2.0);


                    // Creamos una máscara de brillo
                    float glowMask =
                        smoothstep(
                            0.8 * pulse,
                            0.0,
                            dist
                        );


                    // Posición de la luz azul
                    vec2 bluePos =
                        vec2(
                            0.5 + 0.3 * cos(time),
                            0.5 + 0.3 * sin(time * 0.7)
                        );


                    // Distancia de la luz azul
                    float blueDist =
                        length(uv - bluePos);


                    // Intensidad de la luz azul
                    float blueStrength =
                        smoothstep(
                            0.6,
                            0.0,
                            blueDist
                        );


                    // Agregamos el color azul
                    color +=
                        vec3(0.0, 0.3, 0.7)
                        * blueStrength
                        * 0.5;


                    // Posición de la luz verde
                    vec2 greenPos =
                        vec2(
                            0.5 + 0.3 * sin(time * 0.8),
                            0.5 + 0.3 * cos(time * 1.1)
                        );


                    // Distancia de la luz verde
                    float greenDist =
                        length(uv - greenPos);


                    // Intensidad de la luz verde
                    float greenStrength =
                        smoothstep(
                            0.6,
                            0.0,
                            greenDist
                        );


                    // Agregamos el color verde
                    color +=
                        vec3(0.0, 0.8, 0.4)
                        * greenStrength
                        * 0.5;


                    // Agregamos brillo en el centro
                    color +=
                        vec3(0.0, 0.2, 0.3)
                        * glowMask
                        * 0.3;


                    // Creamos una cuadrícula
                    float grid =
                        abs(
                            sin(uv.x * 40.0)
                            * sin(uv.y * 40.0)
                        );


                    // Agregamos la cuadrícula al fondo
                    color +=
                        vec3(0.0, 0.04, 0.06)
                        * pow(grid, 8.0);


                    // Mostramos el color final
                    gl_FragColor =
                        vec4(color, 1.0);
                }
            `;


            // Función para crear un shader
            function cs(type, src) {

                // Creamos el shader
                const s = gl.createShader(type);

                // Le damos el código
                gl.shaderSource(s, src);

                // Compilamos el shader
                gl.compileShader(s);

                // Devolvemos el shader
                return s;
            }


            // Creamos el programa de WebGL
            const prog = gl.createProgram();


            // Agregamos el vertex shader
            gl.attachShader(
                prog,
                cs(gl.VERTEX_SHADER, vs)
            );


            // Agregamos el fragment shader
            gl.attachShader(
                prog,
                cs(gl.FRAGMENT_SHADER, fs)
            );


            // Unimos los shaders
            gl.linkProgram(prog);


            // Utilizamos el programa
            gl.useProgram(prog);


            // Creamos un buffer
            const buf = gl.createBuffer();


            // Seleccionamos el buffer
            gl.bindBuffer(
                gl.ARRAY_BUFFER,
                buf
            );


            // Creamos los puntos del fondo
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


            // Buscamos la posición
            const pos =
                gl.getAttribLocation(
                    prog,
                    'a_position'
                );


            // Activamos la posición
            gl.enableVertexAttribArray(pos);


            // Indicamos cómo leer los datos
            gl.vertexAttribPointer(
                pos,
                2,
                gl.FLOAT,
                false,
                0,
                0
            );


            // Buscamos las variables del shader
            const uTime =
                gl.getUniformLocation(
                    prog,
                    'u_time'
                );

            const uRes =
                gl.getUniformLocation(
                    prog,
                    'u_resolution'
                );

            const uMouse =
                gl.getUniformLocation(
                    prog,
                    'u_mouse'
                );


            // Guardamos la posición del mouse
            let mouse = {
                x: canvas.width / 2,
                y: canvas.height / 2
            };


            // Detectamos el movimiento del mouse
            window.addEventListener(
                'mousemove',
                (event) => {

                    // Obtenemos la posición del canvas
                    const rect =
                        canvas.getBoundingClientRect();


                    // Comprobamos que tenga tamaño
                    if (rect.width && rect.height) {

                        // Calculamos posición X
                        const nx =
                            (event.clientX - rect.left)
                            / rect.width;


                        // Calculamos posición Y
                        const ny =
                            1.0 -
                            (event.clientY - rect.top)
                            / rect.height;


                        // Guardamos la posición
                        mouse.x =
                            nx * canvas.width;

                        mouse.y =
                            ny * canvas.height;
                    }
                }
            );


            // Función que dibuja el fondo
            function render(t) {

                // Ajustamos el tamaño si es necesario
                if (
                    typeof ResizeObserver ===
                    'undefined'
                ) {
                    syncSize();
                }


                // Definimos el área de dibujo
                gl.viewport(
                    0,
                    0,
                    canvas.width,
                    canvas.height
                );


                // Pasamos el tiempo al shader
                if (uTime)
                    gl.uniform1f(
                        uTime,
                        t * 0.001
                    );


                // Pasamos el tamaño
                if (uRes)
                    gl.uniform2f(
                        uRes,
                        canvas.width,
                        canvas.height
                    );


                // Pasamos la posición del mouse
                if (uMouse)
                    gl.uniform2f(
                        uMouse,
                        mouse.x,
                        mouse.y
                    );


                // Dibujamos el fondo
                gl.drawArrays(
                    gl.TRIANGLE_STRIP,
                    0,
                    4
                );


                // Repetimos la animación
                requestAnimationFrame(render);
            }


            // Iniciamos la animación
            render(0);

        })();

    </script>


    <!-- ================================================= -->
    <!-- ENCABEZADO -->
    <!-- ================================================= -->

    <header
        class="flex items-center justify-between px-4 py-4 bg-black/40 backdrop-blur-md border-b border-white/10 sticky top-0 z-10">

        <!-- Parte izquierda del encabezado -->
        <div class="flex items-center gap-3">

            <!-- Icono de sigma -->
            <i
                class="fa-solid fa-sigma text-[#00FF00] text-xl font-bold">
            </i>

            <!-- Título -->
            <h1
                class="text-xl font-bold text-white leading-tight">

                Calculadora de<br>
                Volúmenes

            </h1>

        </div>


        <!-- Botón de configuración -->
        <button class="p-2 text-zinc-400 hover:text-white">

            <i class="fa-solid fa-gear text-lg"></i>

        </button>

    </header>


    <!-- ================================================= -->
    <!-- CONTENIDO PRINCIPAL -->
    <!-- ================================================= -->

    <main
        class="flex-1 p-4 space-y-4 max-w-md mx-auto w-full z-10 relative">


        <!-- ================================================= -->
        <!-- BOTONES DE FIGURAS -->
        <!-- ================================================= -->

        <section
            class="grid grid-cols-2 gap-3"
            id="shape-selectors">


            <!-- Botón del cubo -->
            <button
                class="shape-btn tab-active rounded-xl p-4 flex flex-col items-center justify-center gap-2 relative shadow-sm"
                onclick="selectShape(this, 'cubo')">


                <!-- Punto que indica que está seleccionado -->
                <div
                    class="active-dot absolute top-2 right-2 w-2 h-2 rounded-full bg-[#00FF00] shadow-[0_0_5px_#00FF00]">
                </div>


                <!-- Icono del cubo -->
                <i class="fa-solid fa-cube text-2xl text-[#00FF00]"></i>


                <!-- Nombre -->
                <span class="text-sm font-semibold">
                    Cubo
                </span>

            </button>


            <!-- Botón de esfera -->
            <button
                class="shape-btn tab-inactive rounded-xl p-4 flex flex-col items-center justify-center gap-2 shadow-sm transition-colors hover:bg-black/60 relative"
                onclick="selectShape(this, 'esfera')">

                <i class="fa-regular fa-circle text-2xl"></i>

                <span class="text-sm font-medium">
                    Esfera
                </span>

            </button>


            <!-- Botón de cilindro -->
            <button
                class="shape-btn tab-inactive rounded-xl p-4 flex flex-col items-center justify-center gap-2 shadow-sm transition-colors hover:bg-black/60 relative"
                onclick="selectShape(this, 'cilindro')">

                <i class="fa-solid fa-grip-lines-vertical text-2xl"></i>

                <span class="text-sm font-medium">
                    Cilindro
                </span>

            </button>


            <!-- Botón de cono -->
            <button
                class="shape-btn tab-inactive rounded-xl p-4 flex flex-col items-center justify-center gap-2 shadow-sm transition-colors hover:bg-black/60 relative"
                onclick="selectShape(this, 'cono')">

                <i class="fa-solid fa-caret-up text-3xl"></i>

                <span class="text-sm font-medium">
                    Cono
                </span>

            </button>

        </section>


        <!-- ================================================= -->
        <!-- VISUALIZACIÓN 3D -->
        <!-- ================================================= -->

        <section
            aria-label="Vista 3D de las figuras"
            class="bg-black/40 backdrop-blur-md rounded-2xl border border-white/10 h-64 overflow-hidden relative shadow-sm flex items-center justify-center">


            <!-- Aquí se muestra la figura 3D -->
            <div
                class="w-full h-full"
                id="threejs-container-ANIMATION_4">
            </div>


            <!-- Librería Three.js -->
            <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js">
            </script>


            <script>

                // Obtenemos el contenedor donde estará la figura
                (function() {

                    const container =
                        document.getElementById(
                            'threejs-container-ANIMATION_4'
                        );


                    // Si no existe, detenemos el código
                    if (!container) return;


                    // Obtenemos el ancho
                    const width =
                        container.clientWidth ||
                        window.innerWidth;


                    // Obtenemos el alto
                    const height =
                        container.clientHeight ||
                        window.innerHeight;


                    // Creamos una escena 3D
                    const scene =
                        new THREE.Scene();


                    // Creamos la cámara
                    const camera =
                        new THREE.PerspectiveCamera(
                            75,
                            width / height,
                            0.1,
                            1000
                        );


                    // Creamos el renderizador
                    const renderer =
                        new THREE.WebGLRenderer({
                            alpha: true,
                            antialias: true
                        });


                    // Definimos el tamaño
                    renderer.setSize(
                        width,
                        height
                    );


                    // Mejoramos la calidad según la pantalla
                    renderer.setPixelRatio(
                        window.devicePixelRatio || 1
                    );


                    // Agregamos el canvas 3D
                    container.appendChild(
                        renderer.domElement
                    );


                    // Luz general
                    const ambientLight =
                        new THREE.AmbientLight(
                            0xffffff,
                            0.8
                        );


                    // Agregamos la luz
                    scene.add(ambientLight);


                    // Creamos una luz principal
                    const mainLight =
                        new THREE.DirectionalLight(
                            0xffffff,
                            1
                        );


                    // Posición de la luz
                    mainLight.position.set(
                        5,
                        5,
                        5
                    );


                    // Agregamos la luz
                    scene.add(mainLight);


                    // Luz verde
                    const fillLight =
                        new THREE.DirectionalLight(
                            0x00ff88,
                            0.5
                        );


                    // Posición de la luz verde
                    fillLight.position.set(
                        -5,
                        2,
                        2
                    );


                    // Agregamos la luz
                    scene.add(fillLight);


                    // Material de las figuras
                    const material =
                        new THREE.MeshPhongMaterial({

                            // Color azul
                            color: 0x004a99,

                            // Brillo
                            shininess: 120,

                            // Color del reflejo
                            specular: 0x00ff88,

                            // Permite transparencia
                            transparent: true,

                            // Nivel de transparencia
                            opacity: 0.95

                        });


                    // Creamos las diferentes figuras
                    const geometries = {

                        // Cubo
                        'cubo':
                            new THREE.BoxGeometry(
                                1.8,
                                1.8,
                                1.8
                            ),

                        // Esfera
                        'esfera':
                            new THREE.SphereGeometry(
                                1.2,
                                64,
                                64
                            ),

                        // Cilindro
                        'cilindro':
                            new THREE.CylinderGeometry(
                                1,
                                1,
                                2,
                                64
                            ),

                        // Cono
                        'cono':
                            new THREE.ConeGeometry(
                                1.1,
                                2,
                                64
                            )
                    };


                    // Aquí guardamos la figura actual
                    let currentMesh = null;


                    // Figura seleccionada inicialmente
                    let currentShape = 'cubo';


                    // Función para cambiar la figura
                    function updateShape(shape) {


                        // Si ya existe una figura, la quitamos
                        if (currentMesh) {

                            scene.remove(
                                currentMesh
                            );
                        }


                        // Buscamos la figura seleccionada
                        const geo =
                            geometries[
                                shape.toLowerCase()
                            ] ||
                            geometries['cubo'];


                        // Creamos la nueva figura
                        currentMesh =
                            new THREE.Mesh(
                                geo,
                                material
                            );


                        // Agregamos la figura a la escena
                        scene.add(
                            currentMesh
                        );


                        // Guardamos el nombre
                        currentShape =
                            shape.toLowerCase();
                    }


                    // Mostramos el cubo inicialmente
                    updateShape('cubo');


                    // Alejamos la cámara
                    camera.position.z = 5;


                    // Creamos una función global
                    // para cambiar la figura desde otro código
                    window.updateThreeJSVolumeShape =
                        function(shape) {

                            updateShape(shape);

                        };


                    // Variables para guardar el movimiento del mouse
                    let mouseX = 0;
                    let mouseY = 0;


                    // Detectamos movimiento del mouse
                    window.addEventListener(
                        'mousemove',
                        (event) => {

                            mouseX =
                                (event.clientX /
                                    window.innerWidth) -
                                0.5;


                            mouseY =
                                (event.clientY /
                                    window.innerHeight) -
                                0.5;

                        }
                    );


                    // Función de animación
                    function animate() {


                        // Repetimos la animación
                        requestAnimationFrame(
                            animate
                        );


                        // Comprobamos que exista una figura
                        if (currentMesh) {


                            // Obtenemos el tiempo actual
                            const time =
                                Date.now() * 0.001;


                            // Rotamos la figura
                            currentMesh.rotation.y +=
                                0.015;


                            currentMesh.rotation.x +=
                                0.008;


                            // Creamos un pequeño efecto de tamaño
                            const scaleValue =
                                1.0 +
                                Math.sin(time * 2.0)
                                * 0.1;


                            // Aplicamos el tamaño
                            currentMesh.scale.set(
                                scaleValue,
                                scaleValue,
                                scaleValue
                            );


                            // Hacemos que la figura suba y baje
                            currentMesh.position.y =
                                Math.sin(time) * 0.2;


                            // Hacemos que responda al mouse
                            currentMesh.rotation.y +=
                                (
                                    mouseX * 0.5 -
                                    currentMesh.rotation.y
                                ) * 0.05;


                            currentMesh.rotation.x +=
                                (
                                    mouseY * 0.5 -
                                    currentMesh.rotation.x
                                ) * 0.05;

                        }


                        // Mostramos la escena
                        renderer.render(
                            scene,
                            camera
                        );

                    }


                    // Ajustamos el tamaño cuando cambia la ventana
                    window.addEventListener(
                        'resize',
                        () => {

                            const w =
                                container.clientWidth ||
                                window.innerWidth;

                            const h =
                                container.clientHeight ||
                                window.innerHeight;


                            renderer.setSize(
                                w,
                                h
                            );


                            camera.aspect =
                                w / h;


                            camera.updateProjectionMatrix();

                        }
                    );


                    // Iniciamos la animación
                    animate();

                })();

            </script>

        </section>


        <!-- ================================================= -->
        <!-- PARÁMETROS -->
        <!-- ================================================= -->

        <section
            class="bg-black/40 backdrop-blur-md border border-white/10 rounded-2xl p-5 shadow-sm">


            <!-- Título de la sección -->
            <h2
                class="text-xl font-medium mb-4 text-white">

                Parámetros

            </h2>


            <div class="space-y-4">


                <!-- PRIMER INPUT -->

                <div id="input-group-1">

                    <!-- Texto que indica qué debemos escribir -->
                    <label
                        class="block text-sm font-mono text-zinc-300 mb-1"
                        for="primary-input"
                        id="label-input-1">

                        Lado (a)

                    </label>


                    <div class="relative">

                        <!-- Campo para escribir el número -->
                        <input
                            class="block w-full bg-black/50 border border-white/20 rounded-xl px-4 py-3 font-mono text-lg text-white focus:ring-[#00FF00] focus:border-[#00FF00] shadow-inner-custom"
                            id="primary-input"
                            placeholder="0.00"
                            type="number"
                            value="0.00">


                        <!-- Unidad de medida -->
                        <span
                            class="absolute inset-y-0 right-0 flex items-center pr-4 font-mono text-zinc-400">

                            cm

                        </span>

                    </div>

                </div>


                <!-- SEGUNDO INPUT -->

                <!-- Este campo comienza oculto -->
                <div
                    class="hidden-input"
                    id="input-group-2">


                    <!-- Etiqueta del segundo dato -->
                    <label
                        class="block text-sm font-mono text-zinc-300 mb-1"
                        for="secondary-input"
                        id="label-input-2">

                        Altura (h)

                    </label>


                    <div class="relative">

                        <!-- Segundo campo numérico -->
                        <input
                            class="block w-full bg-black/50 border border-white/20 rounded-xl px-4 py-3 font-mono text-lg text-white focus:ring-[#00FF00] focus:border-[#00FF00] shadow-inner-custom"
                            id="secondary-input"
                            placeholder="0.00"
                            type="number"
                            value="0.00">


                        <!-- Unidad de medida -->
                        <span
                            class="absolute inset-y-0 right-0 flex items-center pr-4 font-mono text-zinc-400">

                            cm

                        </span>

                    </div>

                </div>


                <!-- FÓRMULA -->

                <div
                    class="bg-black/50 rounded-xl p-4 flex justify-between items-center border border-white/10">


                    <!-- Texto -->
                    <span
                        class="font-mono text-sm text-zinc-300">

                        Fórmula:

                    </span>


                    <!-- Fórmula que cambia según la figura -->
                    <span
                        class="font-mono font-medium gradient-text"
                        id="formula-display">

                        V = a³

                    </span>

                </div>


                <!-- BOTONES DE VALORES RÁPIDOS -->

                <div
                    class="flex gap-2"
                    id="quick-actions">


                    <!-- Botón con valor 1 -->
                    <button
                        class="flex-1 bg-black/50 hover:bg-black/70 transition-colors border border-white/10 hover:border-white/30 rounded-full py-2 px-3 text-sm font-mono text-zinc-200"
                        onclick="setQuickValue(1)">

                        a = 1

                    </button>


                    <!-- Botón con valor 5 -->
                    <button
                        class="flex-1 bg-black/50 hover:bg-black/70 transition-colors border border-white/10 hover:border-white/30 rounded-full py-2 px-3 text-sm font-mono text-zinc-200"
                        onclick="setQuickValue(5)">

                        a = 5

                    </button>


                    <!-- Botón con valor 10 -->
                    <button
                        class="flex-1 bg-black/50 hover:bg-black/70 transition-colors border border-white/10 hover:border-white/30 rounded-full py-2 px-3 text-sm font-mono text-zinc-200"
                        onclick="setQuickValue(10)">

                        a = 10

                    </button>

                </div>

            </div>

        </section>


        <!-- ================================================= -->
        <!-- CALCULAR Y RESULTADO -->
        <!-- ================================================= -->

        <section class="space-y-4 mt-2">


            <!-- BOTÓN CALCULAR -->

            <button
                class="w-full btn-gradient font-bold rounded-xl py-4 flex items-center justify-center gap-2 transition-colors shadow-lg"
                onclick="calculateVolume()">


                <!-- Icono de calculadora -->
                <i class="fa-solid fa-calculator text-black"></i>


                <!-- Texto del botón -->
                Calcular Volumen

            </button>


            <!-- RESULTADO -->

            <div
                class="bg-black/40 backdrop-blur-md glow-result rounded-2xl p-6 text-center relative overflow-hidden">


                <!-- Fondo para mejorar el contraste -->
                <div
                    class="absolute inset-0 bg-black/40">
                </div>


                <div class="relative z-10">


                    <!-- Título -->
                    <h3
                        class="text-xs tracking-wider font-mono text-[#007BFF] mb-2 font-semibold">

                        VOLUMEN CALCULADO

                    </h3>


                    <div
                        class="flex items-baseline justify-center gap-1">


                        <!-- Aquí aparece el resultado -->
                        <span
                            class="text-5xl font-bold tracking-tight text-white drop-shadow-[0_0_10px_rgba(255,255,255,0.5)]"
                            id="result-display">

                            0.00

                        </span>


                        <!-- Unidad -->
                        <span
                            class="text-xl font-medium text-[#00FF00]">

                            cm³

                        </span>

                    </div>

                </div>

            </div>

        </section>

    </main>


    <!-- ================================================= -->
    <!-- NAVEGACIÓN INFERIOR -->
    <!-- ================================================= -->

    <nav
        class="fixed bottom-0 left-0 w-full bg-black/40 backdrop-blur-md border-t border-white/10 px-6 py-3 flex justify-between items-center z-20 pb-safe">


        <!-- Botón cubo -->
        <button
            class="nav-active w-12 h-12 flex items-center justify-center flex-col gap-1"
            onclick="document.querySelectorAll('.shape-btn')[0].click()">

            <i class="fa-solid fa-cube text-xl"></i>

            <span class="text-[10px] font-bold leading-none">
                Cubo
            </span>

        </button>


        <!-- Botón esfera -->
        <button
            class="w-12 h-12 flex items-center justify-center text-zinc-400 hover:text-white transition-colors"
            onclick="document.querySelectorAll('.shape-btn')[1].click()">

            <i class="fa-regular fa-circle text-xl"></i>

        </button>


        <!-- Botón cilindro -->
        <button
            class="w-12 h-12 flex items-center justify-center text-zinc-400 hover:text-white transition-colors"
            onclick="document.querySelectorAll('.shape-btn')[2].click()">

            <i class="fa-solid fa-grip-lines-vertical text-xl"></i>

        </button>


        <!-- Botón cono -->
        <button
            class="w-12 h-12 flex items-center justify-center text-zinc-400 hover:text-white transition-colors"
            onclick="document.querySelectorAll('.shape-btn')[3].click()">

            <i class="fa-solid fa-caret-up text-2xl"></i>

        </button>

    </nav>


    <!-- ================================================= -->
    <!-- JAVASCRIPT PRINCIPAL -->
    <!-- ================================================= -->

    <script>


        // Guardamos la figura seleccionada
        // Al comenzar, está seleccionado el cubo
        let currentSelectedShape = 'cubo';


        // =================================================
        // INFORMACIÓN DE CADA FIGURA
        // =================================================

        const shapeData = {


            // Información del cubo
            'cubo': {

                // Nombre del primer dato
                label1: 'Lado (a)',

                // Fórmula del volumen
                formula: 'V = a³',

                // Letra utilizada en los botones rápidos
                quickLabel: 'a',

                // El cubo no necesita altura
                needsHeight: true
            },


            // Información de la esfera
            'esfera': {

                label1: 'Radio (r)',

                formula: 'V = 4/3 π r³',

                quickLabel: 'r',

                // La esfera no necesita altura
                needsHeight: true
            },


            // Información del cilindro
            'cilindro': {

                label1: 'Radio (r)',

                formula: 'V = π r² h',

                quickLabel: 'r',

                // El cilindro sí necesita altura
                needsHeight: true
            },


            // Información del cono
            'cono': {

                label1: 'Radio (r)',

                formula: 'V = 1/3 π r² h',

                quickLabel: 'r',

                // El cono sí necesita altura
                needsHeight: true
            }

        };


        // =================================================
        // CAMBIAR FIGURA
        // =================================================

        // Esta función se ejecuta cuando seleccionamos
        // una figura
        function selectShape(btn, shapeName) {


            // Guardamos la figura seleccionada
            currentSelectedShape = shapeName;


            // Actualizamos la figura 3D
            if (
                typeof window.updateThreeJSVolumeShape ===
                'function'
            ) {

                window.updateThreeJSVolumeShape(
                    shapeName
                );
            }


            // Buscamos el contenedor de los botones
            const container =
                document.getElementById(
                    'shape-selectors'
                );


            // Buscamos todos los botones
            const buttons =
                container.querySelectorAll(
                    '.shape-btn'
                );


            // Recorremos todos los botones
            buttons.forEach(b => {


                // Quitamos el estilo activo
                b.classList.remove(
                    'tab-active'
                );


                // Agregamos el estilo inactivo
                b.classList.add(
                    'tab-inactive',
                    'hover:bg-black/60'
                );


                // Buscamos el icono
                const icon =
                    b.querySelector('i');


                // Si existe el icono
                if (icon) {

                    // Quitamos el color verde
                    icon.classList.remove(
                        'text-[#00FF00]'
                    );
                }


                // Buscamos el texto
                const textSpan =
                    b.querySelector('span');


                // Si existe
                if (textSpan) {

                    // Quitamos texto grueso
                    textSpan.classList.remove(
                        'font-semibold'
                    );

                    // Agregamos texto normal
                    textSpan.classList.add(
                        'font-medium'
                    );
                }


                // Buscamos el punto verde
                const dot =
                    b.querySelector(
                        '.active-dot'
                    );


                // Si existe, lo eliminamos
                if (dot) {

                    dot.remove();
                }

            });


            // =================================================
            // ACTIVAR EL BOTÓN SELECCIONADO
            // =================================================

            // Agregamos estilo activo
            btn.classList.add(
                'tab-active'
            );


            // Quitamos estilo inactivo
            btn.classList.remove(
                'tab-inactive',
                'hover:bg-black/60'
            );


            // Buscamos el icono
            const icon =
                btn.querySelector('i');


            // Lo ponemos verde
            if (icon) {

                icon.classList.add(
                    'text-[#00FF00]'
                );
            }


            // Buscamos el texto
            const textSpan =
                btn.querySelector('span');


            // Ponemos el texto en negrita
            if (textSpan) {

                textSpan.classList.add(
                    'font-semibold'
                );

                textSpan.classList.remove(
                    'font-medium'
                );
            }


            // Creamos el punto verde
            const dot =
                document.createElement('div');


            // Le damos clases al punto
            dot.className =
                'active-dot absolute top-2 right-2 w-2 h-2 rounded-full bg-[#00FF00] shadow-[0_0_5px_#00FF00]';


            // Agregamos el punto al botón
            btn.appendChild(dot);


            // Actualizamos los parámetros
            updateParametersUI(shapeName);


            // Reiniciamos el resultado
            document.getElementById(
                'result-display'
            ).textContent = '0.00';

        }


        // =================================================
        // ACTUALIZAR LOS PARÁMETROS
        // =================================================

        function updateParametersUI(shapeName) {


            // Obtenemos los datos de la figura
            const data =
                shapeData[shapeName];


            // Cambiamos el nombre del primer input
            document.getElementById(
                'label-input-1'
            ).textContent =
                data.label1;


            // Cambiamos la fórmula
            document.getElementById(
                'formula-display'
            ).textContent =
                data.formula;


            // Buscamos el segundo input
            const inputGroup2 =
                document.getElementById(
                    'input-group-2'
                );


            // Revisamos si necesita altura
            if (data.needsHeight) {


                // Mostramos el segundo input
                inputGroup2.classList.remove(
                    'hidden-input'
                );

            } else {


                // Ocultamos el segundo input
                inputGroup2.classList.add(
                    'hidden-input'
                );

            }


            // Buscamos los botones rápidos
            const quickButtons =
                document
                    .getElementById(
                        'quick-actions'
                    )
                    .querySelectorAll(
                        'button'
                    );


            // Cambiamos el texto de los botones
            quickButtons[0].textContent =
                `${data.quickLabel} = 1`;

            quickButtons[1].textContent =
                `${data.quickLabel} = 5`;

            quickButtons[2].textContent =
                `${data.quickLabel} = 10`;

        }


        // =================================================
        // VALORES RÁPIDOS
        // =================================================

        // Esta función coloca un número automáticamente
        // en el primer campo
        function setQuickValue(val) {


            // Buscamos el primer input
            const input =
                document.getElementById(
                    'primary-input'
                );


            // Colocamos el valor
            input.value = val;

        }


        // =================================================
        // CALCULAR VOLUMEN
        // =================================================

        function calculateVolume() {


            // Obtenemos el primer número
            const val1 =
                parseFloat(
                    document.getElementById(
                        'primary-input'
                    ).value
                ) || 0;


            // Obtenemos el segundo número
            const val2 =
                parseFloat(
                    document.getElementById(
                        'secondary-input'
                    ).value
                ) || 0;


            // Creamos una variable para guardar
            // el resultado
            let volume = 0;


            // Revisamos qué figura está seleccionada
            switch (currentSelectedShape) {


                // =================================================
                // CUBO
                // =================================================

                case 'cubo':

                    // Fórmula:
                    // V = lado³
                    volume =
                        Math.pow(
                            val1,
                            3
                        );

                    // Terminamos este caso
                    break;


                // =================================================
                // ESFERA
                // =================================================

                case 'esfera':

                    // Fórmula:
                    // V = 4/3 × π × radio³
                    volume =
                        (4 / 3) *
                        Math.PI *
                        Math.pow(
                            val1,
                            3
                        );

                    break;


                // =================================================
                // CILINDRO
                // =================================================

                case 'cilindro':

                    // Fórmula:
                    // V = π × radio² × altura
                    volume =
                        Math.PI *
                        Math.pow(
                            val1,
                            2
                        ) *
                        val2;

                    break;


                // =================================================
                // CONO
                // =================================================

                case 'cono':

                    // Fórmula:
                    // V = 1/3 × π × radio² × altura
                    volume =
                        (1 / 3) *
                        Math.PI *
                        Math.pow(
                            val1,
                            2
                        ) *
                        val2;

                    break;

            }


            // Mostramos el resultado
            // toFixed(2) deja solamente 2 decimales
            document.getElementById(
                'result-display'
            ).textContent =
                volume.toFixed(2);

        }

    </script>

</body>

</html>

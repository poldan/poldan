<script lang="ts">
  import { onMount } from 'svelte';
  import { spring } from 'svelte/motion'; // <--- IMPORTANTE

  let scrollY = 0;
  let innerWidth = 0;
  let innerHeight = 0; // Necesario para centrar inicial

  // Configuración del movimiento (Spring)
  // stiffness: rigidez (bajo = más suave)
  // damping: amortiguación (bajo = más rebote)
  let coords = spring(
    { x: 0, y: 0 },
    { stiffness: 0.05, damping: 0.25 }
  );

  // Variables reactivas para la animación del Hero
  $: scale = Math.max(0.8, 1 - scrollY / 1000);
  $: opacity = Math.max(0, 1 - scrollY / 600);
  $: heroTranslate = scrollY * 0.5;

  function handleMouseMove(e: MouseEvent) {
    coords.set({ x: e.clientX + 300, y: e.clientY + 300});
  }

  function scrollToTop(e: Event) {
    e.preventDefault();
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
  
  // Centrar la luz inicialmente al cargar
  onMount(() => {
    coords.set({ x: window.innerWidth / 2, y: window.innerHeight / 2 }, { hard: true });
  });
</script>

<svelte:window 
  bind:scrollY 
  bind:innerWidth 
  bind:innerHeight 
  on:mousemove={handleMouseMove} 
/>

<!-- Contenedor Principal -->
<div class="min-h-screen bg-[#030c09] text-white font-sans overflow-x-hidden selection:bg-[#51ad7c] selection:text-white">

  <!-- HEADER (h-20 = 80px de altura) -->
  <header class="fixed top-0 w-full z-50 transition-all duration-300 {scrollY > 50 ? 'bg-[#030c09]/90 backdrop-blur-md border-b border-white/5' : 'bg-transparent'}">
    <div class="max-w-7xl mx-auto px-6 h-20 flex items-center justify-between">
      <!-- Logo -->
      <a href="/" on:click={scrollToTop} class="font-bold text-2xl tracking-tight flex items-center gap-2 cursor-pointer">
        <img
          src="/poldan-logo.png"
          alt="Poldan Design logo"
          class="w-9 h-9 rounded-xl object-cover"
        />
        <span>Poldan Design</span>
      </a>

      <!-- Menú -->
      <nav class="hidden md:flex gap-8 text-sm font-medium text-gray-300">
        <button on:click={scrollToTop} class="hover:text-[#51ad7c] transition-colors cursor-pointer bg-transparent border-none p-0 font-medium text-gray-300">
          Inicio
        </button>
        <a href="#servicios" class="hover:text-[#51ad7c] transition-colors">Servicios</a>
        <a href="#contact" class="hover:text-[#51ad7c] transition-colors">Contacto</a>
        <a href="#about" class="hover:text-[#51ad7c] transition-colors">Nosotros</a>
      </nav>
    </div>
  </header>

  <!-- HERO SECTION -->
  <!-- CAMBIO: Eliminado 'overflow-hidden' para que el degradado no se corte al escalar -->
  <section
    id="home"
    class="relative h-screen flex flex-col items-center justify-center sticky top-0"
    style="transform: translateY({heroTranslate}px); opacity: {opacity}; scale: {scale}; pointer-events: {opacity < 0.1 ? 'none' : 'auto'};"
  >
    <!-- ... resto del contenido igual ... -->
    
    <!-- Opcional: Puedes hacer el blob más responsive para móviles pequeños, 
         cambiando w-[600px] por w-[80vw] md:w-[600px] si quieres, 
         pero quitar el overflow-hidden es lo principal. -->
        <!-- LUZ QUE SIGUE AL MOUSE -->
    <!-- Wrapper: Controla la posición X/Y basada en el mouse -->
    <div 
      class="absolute top-0 left-0 pointer-events-none z-0 will-change-transform"
      style="transform: translate({$coords.x}px, {$coords.y}px);"
    >
      <!-- Blob: Controla el aspecto visual, centrado y el pulso -->
      <!-- Usamos -translate-x/y-1/2 para que el centro del blob coincida con el cursor -->
      <div
        class="w-[600px] h-[600px] bg-[#51ad7c] rounded-full blur-[120px] opacity-20 -translate-x-1/2 -translate-y-1/2 animate-pulse-slow"
      ></div>
    </div>

    <!-- ... resto del contenido ... -->

    <div class="relative z-10 text-center px-4 max-w-4xl">
      <h1 class="text-5xl md:text-7xl font-bold tracking-tight mb-6 leading-tight">
        <!-- Estilo Metálico (Gris-Blanco-Gris) -->
        <span class="bg-clip-text text-transparent bg-gradient-to-r from-gray-500 via-gray-100 to-gray-500">
          Tu negocio merece
        </span>
        <br />
        <!-- Estilo Neón Interno (Verde -> Verde Claro -> Verde) -->
        <span class="bg-clip-text text-transparent bg-gradient-to-r from-[#51ad7c] via-[#a7f3cb] to-[#51ad7c]">
          una web a la altura.
        </span>
      </h1>


      <p class="text-lg md:text-xl text-gray-400 mb-10 max-w-2xl mx-auto font-light">
        Presencia digital diseñada para destacar.
      </p>

      <div class="flex flex-col sm:flex-row gap-4 justify-center items-center">
        <a href="#contact"
          class="px-8 py-3 rounded-full bg-[#2c734e] text-white font-semibold hover:bg-[#2f7a53] hover:scale-105 transition-all duration-300 shadow-[0_0_20px_rgba(61,153,104,0.3)] hover:shadow-[0_0_30px_rgba(61,153,104,0.5)]"
        >
          Conectemos
        </a>
        <a href="#about"
          class="px-8 py-3 rounded-full border border-white/20 text-white font-medium hover:bg-white/10 hover:border-white/40 hover:scale-105 transition-all duration-300 backdrop-blur-sm"
        >
          Saber más
        </a>
      </div>
    </div>
  </section>

  <!-- CONTENEDOR DE SECCIONES -->
  <div class="relative z-20 bg-[#030c09] border-t border-white/10 shadow-[0_-50px_100px_rgba(0,0,0,1)]">
    <div class="mx-auto h-[2px] w-2/5 bg-gradient-to-r from-transparent via-[#51ad7c]/70 to-transparent opacity-70 rounded-full"></div>

    <!-- SECCIÓN SERVICIOS -->
    <section id="servicios" class="min-h-screen flex flex-col justify-center py-20 px-6 max-w-7xl mx-auto scroll-mt-0">
      <h2 class="text-3xl md:text-4xl font-bold mb-16 text-center">Nuestros Servicios</h2>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <!-- Widget 1: Diseño Web -->
        <div class="group bg-white/5 border border-white/5 rounded-3xl p-6 hover:bg-white/10 hover:border-[#51ad7c]/30 transition-all duration-500 cursor-default overflow-hidden relative">
          <div class="h-48 w-full rounded-2xl mb-6 overflow-hidden border border-white/5">
            <img 
              src="/software-design.png" 
              alt="Diseño Web" 
              class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
            />
          </div>
          <h3 class="text-xl font-semibold mb-2 text-white group-hover:text-[#51ad7c] transition-colors">Diseño Web</h3>
          <p class="text-gray-400 text-sm leading-relaxed">Interfaces intuitivas con estética premium que capturan la esencia de tu marca.</p>
        </div>

        <!-- Widget 2: Desarrollo Software -->
        <div class="group bg-white/5 border border-white/5 rounded-3xl p-6 hover:bg-white/10 hover:border-[#51ad7c]/30 transition-all duration-500 cursor-default overflow-hidden relative">
          <div class="h-48 w-full rounded-2xl mb-6 overflow-hidden border border-white/5">
             <img 
              src="/software-design.png" 
              alt="Desarrollo Software" 
              class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
            />
          </div>
          <h3 class="text-xl font-semibold mb-2 text-white group-hover:text-[#51ad7c] transition-colors">Desarrollo Software</h3>
          <p class="text-gray-400 text-sm leading-relaxed">Código limpio, escalable y optimizado utilizando tecnologías de vanguardia como Svelte.</p>
        </div>

        <!-- Widget 3: Consultoría -->
        <div class="group bg-white/5 border border-white/5 rounded-3xl p-6 hover:bg-white/10 hover:border-[#51ad7c]/30 transition-all duration-500 cursor-default overflow-hidden relative">
          <div class="h-48 w-full rounded-2xl mb-6 overflow-hidden border border-white/5">
             <img 
              src="/software-design.png" 
              alt="Consultoría" 
              class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
            />
          </div>
          <h3 class="text-xl font-semibold mb-2 text-white group-hover:text-[#51ad7c] transition-colors">Consultoría</h3>
          <p class="text-gray-400 text-sm leading-relaxed">Estrategia digital y auditoría técnica para llevar tu proyecto al siguiente nivel.</p>
        </div>
      </div>
    </section>

    <!-- CONTACT SECTION -->
    <section id="contact" class="min-h-screen flex flex-col justify-center py-20 px-6 border-t border-white/5 bg-gradient-to-b from-[#030c09] to-[#05100c] scroll-mt-0">
      <div class="max-w-2xl mx-auto text-center w-full">
        <h2 class="text-3xl md:text-4xl font-bold mb-8 text-center">Contacta con nosotros</h2>
        <p class="text-gray-400 mb-10">¿Tienes una idea en mente? Hablemos.</p>

        <form action="https://api.web3forms.com/submit" method="POST" class="space-y-4 text-left">
          <input type="hidden" name="access_key" value="TU-CLAVE-AQUI">
          <input type="checkbox" name="botcheck" class="hidden" style="display: none;">

          <div>
            <label for="email" class="block text-xs uppercase tracking-wider text-gray-500 mb-2 ml-1">Correo electrónico</label>
            <input type="email" name="email" id="email" required placeholder="tu@email.com" class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#51ad7c] focus:ring-1 focus:ring-[#51ad7c] transition-all placeholder-gray-600" />
          </div>

          <div>
            <label for="message" class="block text-xs uppercase tracking-wider text-gray-500 mb-2 ml-1">Mensaje</label>
            <textarea name="message" id="message" rows="4" required placeholder="Cuéntanos sobre tu proyecto..." class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#51ad7c] focus:ring-1 focus:ring-[#51ad7c] transition-all placeholder-gray-600"></textarea>
          </div>

          <button type="submit" class="w-full bg-gradient-to-r from-[#51ad7c] to-emerald-700 hover:brightness-110 text-white font-bold py-4 rounded-xl transition-all shadow-lg transform active:scale-95 mt-4">
            Enviar Mensaje
          </button>
        </form>
      </div>
    </section>

    <!-- SECCIÓN NOSOTROS -->
    <section id="about" class="min-h-screen flex flex-col justify-center py-20 px-6 max-w-7xl mx-auto scroll-mt-0 border-t border-white/5">
      <div class="text-center mb-16">
        <h2 class="text-3xl md:text-4xl font-bold mb-6 text-center">Nosotros</h2>
        <p class="text-xl text-gray-400 max-w-2xl mx-auto">
          Una alianza estratégica entre Barcelona y Estados Unidos.
        </p>
      </div>

      <!-- Usamos items-stretch para igualar alturas -->
      <div class="grid grid-cols-1 md:grid-cols-2 gap-12 items-stretch">
        
        <!-- Texto -->
        <div class="space-y-6 flex flex-col justify-center">
          <h3 class="text-2xl font-bold text-white">Tecnología y Negocio</h3>
          <p class="text-gray-400 leading-relaxed">
            Somos una agencia formada por dos socios que combinan la excelencia técnica con la visión empresarial. Operamos de forma remota entre España y EE.UU., aportando una perspectiva global a cada proyecto.
          </p>
          
          <div class="mt-8 space-y-4">
            <!-- Perfil 1 -->
            <div class="bg-white/5 p-5 rounded-2xl border border-white/5">
              <div class="font-bold text-white text-lg">Ingeniería y Desarrollo</div>
              <div class="text-xs text-[#51ad7c] tracking-wider mb-2">Daniel</div>
              <p class="text-sm text-gray-400">
                Graduado en Ingeniería Informática por la <strong>Universidad Autónoma de Barcelona</strong> y la <strong>University of Montana</strong>. Especializado en desarrollo web de alto rendimiento.
              </p>
            </div>

            <!-- Perfil 2 -->
            <div class="bg-white/5 p-5 rounded-2xl border border-white/5">
              <div class="font-bold text-white text-lg">Estrategia y Negocio</div>
              <div class="text-xs text-[#51ad7c] tracking-wider mb-2">Pol</div>
              <p class="text-sm text-gray-400">
                Estudiante de Administración y Dirección de Empresas (ADE) en la <strong>Universidad Autónoma de Barcelona</strong>. Enfocado en la rentabilidad del producto y análisis de mercado.
              </p>
            </div>
          </div>
        </div>

                <!-- Mapa Global Estilizado -->
        <div class="bg-[#05100c] rounded-3xl border border-white/10 relative overflow-hidden group h-full min-h-[400px]">
            
            <!-- 1. Imagen de fondo (Puntos) - AHORA VA PRIMERO para estar al fondo -->
            <img
              src="/world-map-dots.png" 
              class="absolute top-0 left-0 w-full h-full object-cover opacity-40 mix-blend-screen pointer-events-none z-0"
              alt="World Map"
            />

            <!-- 2. Fondo Mapa SVG + Puntos integrados - AHORA VA DESPUÉS (z-10) -->
            <div class="absolute inset-0 w-full h-full z-10">
                
                <!-- preserveAspectRatio="xMidYMid slice" asegura que el mapa llene todo (cover) -->
                <svg viewBox="0 0 1000 500" preserveAspectRatio="xMidYMid slice" fill="none" xmlns="http://www.w3.org/2000/svg" class="w-full h-full">
                    <!-- Continentes -->
                    <path d="M-50,50 Q150,20 300,80 T500,200 T400,450 T150,500 T-50,300 Z" fill="#51ad7c" fill-opacity="0.05" stroke="#51ad7c" stroke-opacity="0.1" />
                    <path d="M600,50 Q800,20 950,100 T1050,400 T750,500 T550,300 Z" fill="#51ad7c" fill-opacity="0.05" stroke="#51ad7c" stroke-opacity="0.1" />
                    <!-- Línea de conexión -->
                    <path 
                      d="M345,208 Q470,120 595,213" 
                      stroke="#51ad7c" 
                      stroke-width="2" 
                      stroke-dasharray="6 6" 
                      stroke-opacity="0.6" 
                      fill="none"
                      class="animate-dash" 
                    />

                    <!-- PUNTO MONTANA (Coordenada 200, 185) -->
                    <!-- Añado z-index alto aquí visualmente mediante orden de renderizado SVG -->
                    <foreignObject x="245" y="200" width="200" height="200" style="overflow: visible; z-index: 50;">
                      <div xmlns="http://www.w3.org/1999/xhtml" class="flex flex-col items-center group/marker relative z-50">
                        <div class="relative">
                          <div class="w-4 h-4 bg-[#51ad7c] rounded-full animate-pulse shadow-[0_0_15px_rgba(81,173,124,0.8)]"></div>
                          <div class="absolute -inset-4 border border-[#51ad7c]/30 rounded-full scale-0 group-hover/marker:scale-100 transition-transform duration-500"></div>
                        </div>
                        <!-- Añado z-50 explícito al texto -->
                        <div class="mt-3 text-center backdrop-blur-md bg-black/80 px-4 py-1.5 rounded-xl border border-white/20 shadow-2xl z-50 relative">
                          <div class="font-bold text-white text-sm shadow-black drop-shadow-md">Montana</div>
                          <div class="text-[10px] text-gray-300 uppercase tracking-wider font-semibold">EE. UU.</div>
                        </div>
                      </div>
                    </foreignObject>

                    <!-- PUNTO BARCELONA (Coordenada 800, 210) -->
                    <foreignObject x="495" y="205" width="200" height="200" style="overflow: visible; z-index: 50;">
                      <div xmlns="http://www.w3.org/1999/xhtml" class="flex flex-col items-center group/marker relative z-50">
                        <div class="relative">
                          <div class="w-4 h-4 bg-[#51ad7c] rounded-full animate-pulse delay-700 shadow-[0_0_15px_rgba(81,173,124,0.8)]"></div>
                          <div class="absolute -inset-4 border border-[#51ad7c]/30 rounded-full scale-0 group-hover/marker:scale-100 transition-transform duration-500"></div>
                        </div>
                        <!-- Añado z-50 explícito al texto -->
                        <div class="mt-3 text-center backdrop-blur-md bg-black/80 px-4 py-1.5 rounded-xl border border-white/20 shadow-2xl z-50 relative">
                          <div class="font-bold text-white text-sm shadow-black drop-shadow-md">Barcelona</div>
                          <div class="text-[10px] text-gray-300 uppercase tracking-wider font-semibold">España</div>
                        </div>
                      </div>
                    </foreignObject>

                </svg>
            </div>

            <!-- Texto central inferior -->
            <div class="absolute bottom-8 left-0 right-0 text-center px-4 z-20 pointer-events-none">
                <p class="text-sm text-gray-400 max-w-md mx-auto backdrop-blur-sm py-2 px-4 rounded-full bg-black/20 border border-white/5">
                    Conectando talento global para resultados locales.
                </p>
            </div>
        </div>
      </div>
    </section>

    <!-- FOOTER -->
    <footer class="py-12 px-6 border-t border-white/5 text-center md:text-left bg-[#020202]">
      <div class="max-w-7xl mx-auto flex flex-col md:flex-row justify-between items-center gap-6">
        <div>
          <div class="font-bold text-xl mb-2 flex items-center justify-center md:justify-start gap-2">
            <img src="/poldan-logo.png" alt="Poldan Design Logo" class="w-8 h-8 rounded-lg object-cover" />
            Poldan Design
          </div>
          <p class="text-gray-500 text-sm">
            © {new Date().getFullYear()} Poldan Design. Missoula & Barcelona.
          </p>
        </div>

        <div class="text-gray-400 text-sm flex flex-col items-center md:items-end">
          <p>Missoula, Montana</p>
          <p>775 Wyoming St.</p>
          <p class="text-[#51ad7c] mt-2">hello@poldandesign.com</p>
        </div>
      </div>
    </footer>
  </div>
</div>

<style>
  :global(html) {
    scroll-behavior: smooth;
  }

  @keyframes pulse-slow {
    0%, 100% { opacity: 0.15; transform: translate(-50%, -50%) scale(1); }
    50% { opacity: 0.25; transform: translate(-50%, -50%) scale(1.1); }
  }
  .animate-pulse-slow { animation: pulse-slow 8s ease-in-out infinite; }

    @keyframes dash {
    to {
      stroke-dashoffset: -12; /* Debe ser igual a la suma del dasharray (6+6) */
    }
  }
  
  .animate-dash {
    animation: dash 1s linear infinite;
  }

</style>

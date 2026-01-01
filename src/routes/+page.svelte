<script lang="ts">
  import { onMount } from 'svelte';
  import { spring } from 'svelte/motion';
  import { fade, fly } from 'svelte/transition'; // Asegúrate de tener esta importación

  // Estado del formulario
  let formStatus = 'idle'; // 'idle', 'submitting', 'success', 'error'
  let showNotification = false;

  async function handleSubmit(e: any) {
    const formData = new FormData(e.target);
    
    // Tu clave real
    formData.append("access_key", "6057433d-2df7-4e61-a44d-3d2071d7dc51");

    formStatus = 'submitting';

    try {
      const response = await fetch("https://api.web3forms.com/submit", {
        method: "POST",
        body: formData
      });

      const data = await response.json();

      if (data.success) {
        formStatus = 'success';
        showNotification = true;
        e.target.reset(); // Limpia el formulario
        
        // Ocultar notificación automáticamente después de 5 segundos
        setTimeout(() => {
          showNotification = false;
          formStatus = 'idle';
        }, 5000);
      } else {
        formStatus = 'error';
      }
    } catch (err) {
      console.error(err);
      formStatus = 'error';
    }
  }

  let scrollY = 0;
  let innerWidth = 0;
  let innerHeight = 0;
  
  // Estado para controlar la "página" activa
  let currentPath = 'home'; // 'home' | 'about'
  
  // Variable nueva para guardar dónde estaba el usuario
  let lastHomeScroll = 0;

  // Configuración del movimiento (Spring)
  let coords = spring(
    { x: 0, y: 0 },
    { stiffness: 0.05, damping: 0.25 }
  );

  $: scale = Math.max(0.8, 1 - scrollY / 1000);
  $: heroTranslate = scrollY * 0.5;

  function handleMouseMove(e: MouseEvent) {
    coords.set({ x: e.clientX + 200, y: e.clientY + 200});
  }

  // Función de navegación mejorada
  function navigateTo(path: string, sectionId: string | null = null) {
    
    if (path === 'about') {
      // 1. Guardamos la posición actual antes de irnos
      lastHomeScroll = scrollY;
      currentPath = 'about';
      // Reseteamos el scroll al top para la página "About" instantáneamente
      setTimeout(() => window.scrollTo({ top: 0, behavior: 'instant' }), 0);
      
    } else if (path === 'home') {
      currentPath = 'home';
      
      setTimeout(() => {
        if (sectionId) {
          // Si es navegación a una sección (ej: menú), scroll suave
          const el = document.getElementById(sectionId);
          if (el) el.scrollIntoView({ behavior: 'smooth' });
        } else {
          // 2. Si es "volver atrás", restauramos la posición exacta instantáneamente
          window.scrollTo({ top: lastHomeScroll, behavior: 'instant' });
        }
      }, 10); // Pequeño delay para dejar que Svelte renderice el DOM
    }
  }
  
  onMount(() => {
    coords.set({ x: window.innerWidth / 2, y: window.innerHeight / 2 }, { hard: true });
  });

    let currentPlanIndex = 0;
  const totalPlans = 3;

  function prevPlan() {
    currentPlanIndex = (currentPlanIndex - 1 + totalPlans) % totalPlans;
  }

  function nextPlan() {
    currentPlanIndex = (currentPlanIndex + 1) % totalPlans;
  }

  let currentFaqIndex = 0;
  const totalFaqs = 4;

  function prevFaq() {
    if (currentFaqIndex > 0) currentFaqIndex = currentFaqIndex - 1;
  }

  function nextFaq() {
    if (currentFaqIndex < totalFaqs - 1) currentFaqIndex = currentFaqIndex + 1;
  }


</script>

<svelte:window 
  bind:scrollY 
  bind:innerWidth 
  bind:innerHeight 
  on:mousemove={handleMouseMove} 
/>

<!-- Contenedor Principal -->
<div class="min-h-screen bg-[#030c09] text-white font-sans overflow-x-hidden selection:bg-[#51ad7c] selection:text-white">

  <!-- HEADER -->
  <header class="fixed top-0 w-full z-50 transition-all duration-300 {scrollY > 50 ? 'bg-[#030c09]/90 backdrop-blur-md border-b border-white/5 h-20' : 'bg-transparent h-24'} flex items-center">
    <div class="max-w-7xl mx-auto px-6 w-full flex items-center justify-between">
      <!-- Logo -->
      <a href="/" on:click|preventDefault={() => navigateTo('home')} class="font-bold text-2xl tracking-tight flex items-center gap-2 cursor-pointer">
        <img
          src="/poldan-logo.png"
          alt="Poldan Design logo"
          class="w-9 h-9 rounded-xl object-cover"
        />
        <span>Poldan Design</span>
      </a>

      <!-- Menú -->
      <nav class="hidden md:flex items-center gap-8 text-sm font-medium text-gray-300">
        <!-- En home navega a secciones, en about vuelve a home y busca la sección -->
        <a href="#proceso" on:click|preventDefault={() => navigateTo('home', 'proceso')} class="hover:text-[#51ad7c] transition-colors">Proceso</a>
        <a href="#planes" on:click|preventDefault={() => navigateTo('home', 'planes')} class="hover:text-[#51ad7c] transition-colors">Planes</a>
        
        <!-- Botón Nosotros con LÓGICA DE RETORNO -->
        <!-- Si currentPath es 'about', vuelve a 'home'. Si no, va a 'about'. -->
        <a 
          href="#about" 
          on:click|preventDefault={() => currentPath === 'about' ? navigateTo('home') : navigateTo('about')} 
          class="hover:text-[#51ad7c] transition-colors cursor-pointer {currentPath === 'about' ? 'hover:text-[#51ad7c]' : ''}"
        >
          Nosotros
        </a>
        
        <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="hover:text-[#51ad7c] transition-colors">Contacto</a>
      </nav>
    </div>
  </header>

  <!-- CONTENIDO HOME -->
  {#if currentPath === 'home'}
    <!-- Aplicamos transición fade rápida (duration 300ms) -->
    <div in:fade={{ duration: 300 }} out:fade={{ duration: 200 }}>
      
      <!-- HERO SECTION -->
      <section
        id="home"
        class="relative h-screen flex flex-col items-center justify-center sticky top-0"
        style="transform: translateY({heroTranslate}px); scale: {scale};" 
      >
        <!-- LUZ QUE SIGUE AL MOUSE -->
        <div 
          class="absolute top-0 left-0 pointer-events-none z-0 will-change-transform"
          style="transform: translate({$coords.x}px, {$coords.y}px);"
        >
          <div
            class="w-[400px] h-[400px] bg-[#51ad7c] rounded-full blur-[120px] opacity-10 -translate-x-1/2 -translate-y-1/2 animate-pulse-slow"
          ></div>
        </div>

        <div class="relative z-10 text-center px-4 max-w-4xl">
          <h1 class="text-4xl md:text-7xl font-bold tracking-tight mb-6 leading-tight">
            <span class="bg-clip-text text-transparent bg-gradient-to-r from-gray-500 via-gray-100 to-gray-500">
              Tu negocio merece
            </span>
            <br />
            <span class="bg-clip-text text-transparent bg-gradient-to-r from-[#51ad7c] via-[#a7f3cb] to-[#51ad7c]">
              una web a la altura.
            </span>
          </h1>


          <p class="text-lg md:text-xl text-gray-400 mb-10 max-w-2xl mx-auto font-light">
            Presencia digital diseñada para destacar.
          </p>

          <div class="flex flex-col sm:flex-row gap-4 justify-center items-center">
            <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')}
              class="px-8 py-3 rounded-full bg-[#2c734e] text-white font-semibold hover:bg-[#2f7a53] hover:scale-105 transition-all duration-300 shadow-[0_0_20px_rgba(61,153,104,0.3)] hover:shadow-[0_0_30px_rgba(61,153,104,0.5)]"
            >
              Conectemos
            </a>
            <a href="#planes" on:click|preventDefault={() => navigateTo('home', 'planes')}
              class="px-8 py-3 rounded-full border border-white/20 text-white font-medium hover:bg-white/10 hover:border-white/40 hover:scale-105 transition-all duration-300 backdrop-blur-sm"
            >
              Ver Planes
            </a>
          </div>
        </div>
      </section>

      <!-- CONTENEDOR DE SECCIONES -->
      <div class="relative z-20 bg-[#030c09] border-t border-white/10 shadow-[0_-1000px_1000px_rgba(0,0,0,0.1)]">
        
        <!-- Divisor Decorativo -->
        <div class="absolute top-0 left-1/2 transform -translate-x-1/2 h-[2px] w-2/5 bg-gradient-to-r from-transparent via-[#51ad7c]/70 to-transparent opacity-70 rounded-full z-30"></div>

        <!-- SECCIÓN PROCESO -->
        <section id="proceso" class="py-24 px-6 max-w-7xl mx-auto scroll-mt-20 relative">
          <h2 class="text-3xl md:text-4xl font-bold mb-16 text-center">Nuestro proceso</h2>

          <div class="relative">
            <!-- LÍNEA CONECTORA (VERSIÓN ESCRITORIO) - INTACTA -->
            <div class="hidden md:block absolute top-8 left-0 w-full h-0.5 bg-gradient-to-r from-transparent via-white/20 to-transparent z-0 overflow-hidden">
              <div class="absolute top-0 left-0 h-full w-24 bg-gradient-to-r from-transparent via-[#51ad7c] to-transparent animate-scan"></div>
            </div>

            <!-- CONTENEDOR PRINCIPAL: Flex en móvil (scroll), Grid en escritorio -->
            <!-- Se añaden clases para ocultar scrollbar: scrollbar-hide o estilo en línea si no tienes el plugin -->
            <div class="flex overflow-x-auto snap-x snap-mandatory pb-12 gap-8 -mx-6 px-6 md:grid md:grid-cols-4 md:gap-8 md:overflow-visible md:pb-0 md:mx-0 md:px-0 relative z-10 [scrollbar-width:none] [-ms-overflow-style:none] [&::-webkit-scrollbar]:hidden">
              
              <!-- LÍNEA CONECTORA (VERSIÓN MÓVIL) -->
              <!-- Esta línea se mueve con el scroll para conectar los puntos en móvil -->
              <!-- LÍNEA CONECTORA (VERSIÓN MÓVIL) -->
              <div class="md:hidden absolute top-8 left-6 right-6 h-0.5 -z-10 w-[364%] overflow-hidden">
                <!-- fibra con desvanecido hacia los extremos -->
                <div class="relative w-full h-full bg-gradient-to-r from-transparent via-white/20 to-transparent">
                  <!-- led que se mueve por la fibra -->
                  <div class="absolute top-0 left-0 h-full w-24 bg-gradient-to-r from-transparent via-[#51ad7c] to-transparent animate-scan"></div>
                </div>
              </div>
              
              <!-- Paso 1 -->
              <div class="group min-w-[85vw] md:min-w-0 snap-center md:snap-align-none pt-2">
                <div class="relative z-10 bg-[#040f0b] border border-[#51ad7c]/30 w-16 h-16 rounded-2xl flex items-center justify-center mx-auto mb-6 group-hover:scale-110 transition-transform duration-300 shadow-[0_0_15px_rgba(81,173,124,0.2)]">
                  <span class="text-2xl font-bold text-[#51ad7c]">1</span>
                </div>
                <h3 class="text-xl font-bold text-white text-center mb-3">Descubrimiento</h3>
                <p class="text-sm text-gray-400 text-center text-justify px-2 md:px-0">
                  Auditamos tu situación actual. Entendemos tu negocio, tu competencia y tus objetivos para definir el camino correcto.
                </p>
              </div>

              <!-- Paso 2 -->
              <div class="group min-w-[85vw] md:min-w-0 snap-center md:snap-align-none pt-2">
                <div class="relative z-10 bg-[#040f0b] border border-[#51ad7c]/30 w-16 h-16 rounded-2xl flex items-center justify-center mx-auto mb-6 group-hover:scale-110 transition-transform duration-300 shadow-[0_0_15px_rgba(81,173,124,0.2)]">
                  <span class="text-2xl font-bold text-[#51ad7c]">2</span>
                </div>
                <h3 class="text-xl font-bold text-white text-center mb-3">Diseño</h3>
                <p class="text-sm text-gray-400 text-center text-justify px-2 md:px-0">
                  Creamos la identidad visual. Prototipamos la experiencia de usuario antes de escribir código.
                </p>
              </div>

              <!-- Paso 3 -->
              <div class="group min-w-[85vw] md:min-w-0 snap-center md:snap-align-none pt-2">
                <div class="relative z-10 bg-[#040f0b] border border-[#51ad7c]/30 w-16 h-16 rounded-2xl flex items-center justify-center mx-auto mb-6 group-hover:scale-110 transition-transform duration-300 shadow-[0_0_15px_rgba(81,173,124,0.2)]">
                  <span class="text-2xl font-bold text-[#51ad7c]">3</span>
                </div>
                <h3 class="text-xl font-bold text-white text-center mb-3">Desarrollo</h3>
                <p class="text-sm text-gray-400 text-center text-justify px-2 md:px-0">
                  Programación web a medida. Implementamos tu sitio web optimizando velocidad, SEO y compatibilidad con móvil.
                </p>
              </div>

              <!-- Paso 4 -->
              <div class="group min-w-[85vw] md:min-w-0 snap-center md:snap-align-none pt-2">
                <div class="relative z-10 bg-[#040f0b] border border-[#51ad7c]/30 w-16 h-16 rounded-2xl flex items-center justify-center mx-auto mb-6 group-hover:scale-110 transition-transform duration-300 shadow-[0_0_15px_rgba(81,173,124,0.2)]">
                  <span class="text-2xl font-bold text-[#51ad7c]">4</span>
                </div>
                <h3 class="text-xl font-bold text-white text-center mb-3">Entrega y Escala</h3>
                <p class="text-sm text-gray-400 text-center text-justify px-2 md:px-0">
                  Lanzamiento oficial de tu web. Te entregamos las llaves y te enseñamos a gestionar tu nuevo activo digital.
                </p>
              </div>

            </div>
          </div>
        </section>


        <!-- SECCIÓN PLANES -->
        <section id="planes" class="py-24 px-6 max-w-7xl mx-auto border-t border-white/5 scroll-mt-20">
          <div class="text-center mb-16">
              <h2 class="text-3xl md:text-4xl font-bold mb-6">Planes</h2>
              <p class="text-gray-400 max-w-2xl mx-auto">Soluciones escalables para tu negocio. Reembolso completo si no queda satisfecho.</p>
          </div>

          <!-- DESKTOP / TABLET (igual que antes) -->
          <div class="hidden md:grid md:grid-cols-3 gap-8 items-start">
              
              <!-- PLAN 1: STARTER -->
              <div class="bg-white/5 border border-white/10 rounded-3xl p-8 hover:border-[#51ad7c]/30 transition-all duration-300 flex flex-col h-full group">
                  <div class="mb-6">
                      <h3 class="text-xl font-bold text-white mb-2">Esencial</h3>
                      <p class="text-sm text-gray-400">Web funcional, para empezar con presencia online.</p>
                  </div>
                  
                  <ul class="space-y-4 mb-8 flex-grow">
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Página de presentación</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Diseño 100% responsive</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Web informativa básica</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Identidad de marca visible</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Hosting + dominio (1er año)</span>
                      </li>
                  </ul>

                  <div class="pt-6 border-t border-white/10">
                      <p class="text-xs text-[#51ad7c] font-medium italic">
                          La entrada al mundo digital
                      </p>
                      <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="mt-4 block w-full py-3 rounded-xl border border-white/20 text-center text-sm font-semibold hover:bg-white/10 transition-colors">
                          Consultar Disponibilidad
                      </a>
                  </div>
              </div>

              <!-- PLAN 2: BUSINESS (DESTACADO) -->
              <div class="relative bg-[#05100c] border-2 border-[#51ad7c] rounded-3xl p-8 shadow-[0_0_40px_rgba(81,173,124,0.15)] flex flex-col h-full transform md:-translate-y-4">
                  <div class="absolute -top-[2px] -right-[2px] bg-[#51ad7c] text-[#030c09] text-xs font-bold px-4 py-1.5 rounded-bl-xl rounded-tr-[22px]">
                      MÁS VENDIDO
                  </div>
                  
                  <div class="mb-6">
                      <h3 class="text-xl font-bold text-white mb-2">Plus</h3>
                      <p class="text-sm text-gray-400">Plan con la mejor relación calidad-precio.</p>
                  </div>
                  
                  <ul class="space-y-4 mb-8 flex-grow">
                      <li class="flex items-start gap-3 text-sm text-white font-medium">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>5–6 páginas completas</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-white font-medium">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Integración de formularios</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Optimizado para convertir</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>SEO completo</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Hosting + dominio (1er año)</span>
                      </li>
                  </ul>

                  <div class="pt-6 border-t border-white/10">
                      <p class="text-xs text-[#51ad7c] font-medium italic">
                          El impulso que tu marca necesita
                      </p>
                      <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="mt-4 block w-full py-3 rounded-xl bg-[#51ad7c] text-[#030c09] text-center text-sm font-bold hover:brightness-110 transition-all shadow-lg hover:shadow-[#51ad7c]/20">
                          Empezar Ahora
                      </a>
                  </div>
              </div>

              <!-- PLAN 3: PRO -->
              <div class="bg-white/5 border border-white/10 rounded-3xl p-8 hover:border-[#51ad7c]/30 transition-all duration-300 flex flex-col h-full group">
                  <div class="mb-6">
                      <h3 class="text-xl font-bold text-white mb-2">Élite</h3>
                      <p class="text-sm text-gray-400">Lo mejor en diseño, estrategia y resultados.</p>
                  </div>
                  
                  <ul class="space-y-4 mb-8 flex-grow">
                      <li class="flex items-start gap-3 text-sm text-white font-medium">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Todo lo anterior</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Sesión de fotos profesional</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Estrategia digital</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>5 rondas de cambios</span>
                      </li>
                      <li class="flex items-start gap-3 text-sm text-gray-300">
                          <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                          <span>Soporte prioritario</span>
                      </li>
                  </ul>

                  <div class="pt-6 border-t border-white/10">
                      <p class="text-xs text-[#51ad7c] font-medium italic">
                          Para marcas que no se conforman 
                      </p>
                      <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="mt-4 block w-full py-3 rounded-xl border border-white/20 text-center text-sm font-semibold hover:bg-white/10 transition-colors">
                          Consultar Disponibilidad
                      </a>
                  </div>
              </div>
          </div>

          <!-- MOBILE SLIDESHOW -->
          <div class="relative md:hidden">
            <!-- Flecha izquierda -->
            <button
              type="button"
              on:click={prevPlan}
              class="absolute left-2 top-1/2 -translate-y-1/2 z-10 text-white disabled:opacity-30 disabled:cursor-default"
              aria-label="Plan anterior"
              disabled={currentPlanIndex === 0}
            >
              <svg class="w-7 h-7" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
              </svg>
            </button>

            <!-- Flecha derecha -->
            <button
              type="button"
              on:click={nextPlan}
              class="absolute right-2 top-1/2 -translate-y-1/2 z-10 text-white disabled:opacity-30 disabled:cursor-default"
              aria-label="Siguiente plan"
              disabled={currentPlanIndex === totalPlans - 1}
            >
              <svg class="w-7 h-7" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </button>

            <!-- Contenedor de slide -->
            <div class="overflow-hidden">
              <div
                class="flex transition-transform duration-300"
                style={`transform: translateX(-${currentPlanIndex * 100}%);`}
              >
                <!-- Slide 1 -->
                <div class="w-full flex justify-center px-10 flex-shrink-0">
                  <div class="w-full max-w-xs bg-white/5 border border-white/10 rounded-3xl p-8 flex flex-col h-full group">
                      <div class="mb-6">
                          <h3 class="text-xl font-bold text-white mb-2">Esencial</h3>
                          <p class="text-sm text-gray-400">Web funcional, para empezar con presencia online.</p>
                      </div>
                      
                      <ul class="space-y-4 mb-8 flex-grow">
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Página de presentación</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Diseño 100% responsive</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Web informativa básica</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Identidad de marca visible</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Hosting + dominio (1er año)</span>
                          </li>
                      </ul>

                      <div class="pt-6 border-t border-white/10">
                          <p class="text-xs text-[#51ad7c] font-medium italic">
                              La entrada al mundo digital
                          </p>
                          <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="mt-4 block w-full py-3 rounded-xl border border-white/20 text-center text-sm font-semibold hover:bg-white/10 transition-colors">
                              Consultar Disponibilidad
                          </a>
                      </div>
                  </div>
                </div>

                <!-- Slide 2 -->
                <div class="w-full flex justify-center px-10 flex-shrink-0">
                  <div class="w-full max-w-xs relative bg-[#05100c] border-2 border-[#51ad7c] rounded-3xl p-8 shadow-[0_0_40px_rgba(81,173,124,0.15)] flex flex-col h-full">
                      <div class="absolute -top-[2px] -right-[2px] bg-[#51ad7c] text-[#030c09] text-xs font-bold px-4 py-1.5 rounded-bl-xl rounded-tr-[22px]">
                          MÁS VENDIDO
                      </div>
                      
                      <div class="mb-6">
                          <h3 class="text-xl font-bold text-white mb-2">Plus</h3>
                          <p class="text-sm text-gray-400">Plan con la mejor relación calidad-precio.</p>
                      </div>
                      
                      <ul class="space-y-4 mb-8 flex-grow">
                          <li class="flex items-start gap-3 text-sm text-white font-medium">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>5–6 páginas completas</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-white font-medium">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Integración de formularios</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Optimizado para convertir</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>SEO completo</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Hosting + dominio (1er año)</span>
                          </li>
                      </ul>

                      <div class="pt-6 border-t border-white/10">
                          <p class="text-xs text-[#51ad7c] font-medium italic">
                              El impulso que tu marca necesita
                          </p>
                          <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="mt-4 block w-full py-3 rounded-xl bg-[#51ad7c] text-[#030c09] text-center text-sm font-bold hover:brightness-110 transition-all shadow-lg hover:shadow-[#51ad7c]/20">
                              Empezar Ahora
                          </a>
                      </div>
                  </div>
                </div>

                <!-- Slide 3 -->
                <div class="w-full flex justify-center px-10 flex-shrink-0">
                  <div class="w-full max-w-xs bg-white/5 border border-white/10 rounded-3xl p-8 flex flex-col h-full group">
                      <div class="mb-6">
                          <h3 class="text-xl font-bold text-white mb-2">Élite</h3>
                          <p class="text-sm text-gray-400">Lo mejor en diseño, estrategia y resultados.</p>
                      </div>
                      
                      <ul class="space-y-4 mb-8 flex-grow">
                          <li class="flex items-start gap-3 text-sm text-white font-medium">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Todo lo anterior</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Sesión de fotos profesional</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Estrategia digital</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>5 rondas de cambios</span>
                          </li>
                          <li class="flex items-start gap-3 text-sm text-gray-300">
                              <svg class="w-5 h-5 text-[#51ad7c] flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/></svg>
                              <span>Soporte prioritario</span>
                          </li>
                      </ul>

                      <div class="pt-6 border-t border-white/10">
                          <p class="text-xs text-[#51ad7c] font-medium italic">
                              Para marcas que no se conforman 
                          </p>
                          <a href="#contact" on:click|preventDefault={() => navigateTo('home', 'contact')} class="mt-4 block w-full py-3 rounded-xl border border-white/20 text-center text-sm font-semibold hover:bg-white/10 transition-colors">
                              Consultar Disponibilidad
                          </a>
                      </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </section>

        <!-- SECCIÓN FAQs -->
        <section class="py-25 px-6 max-w-4xl mx-auto border-t border-white/5">
            <h2 class="text-3xl md:text-4xl font-bold mb-12 text-center">Preguntas frecuentes</h2>

            <!-- DESKTOP / TABLET: igual que antes -->
            <div class="hidden md:grid grid-cols-1 md:grid-cols-2 gap-6">
                <div class="bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                    <h4 class="font-bold text-white mb-2 text-lg">¿Cuánto tardáis en entregar?</h4>
                    <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                        Entregamos webs corporativas estándar en 2 a 4 semanas tras recibir todo el material. Proyectos complejos se planifican con un plazo ajustado a cada caso.
                    </p>
                </div>
                <div class="bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                    <h4 class="font-bold text-white mb-2 text-lg">¿Qué incluye el servicio?</h4>
                    <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                        Incluye diseño y desarrollo a medida, dominio, hosting optimizado, mantenimiento técnico y la entrega total de la propiedad intelectual del proyecto.
                    </p>
                </div>
                <div class="bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                    <h4 class="font-bold text-white mb-2 text-lg">¿Por qué elegir Poldan?</h4>
                    <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                        Diseñamos y construimos activos digitales, mejorando tu presencia online con SEO, arquitectura moderna y experiencia de usuario optimizada para tus objetivos.
                    </p>
                </div>
                <div class="bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                    <h4 class="font-bold text-white mb-2 text-lg">¿Tengo que pagar todo ya?</h4>
                    <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                        No. Trabajamos con 50% al inicio y 50% al finalizar. Así aseguramos seguridad y que estés 100% satisfecho con el resultado antes de pagar lo restante.
                    </p>
                </div>
            </div>

            <!-- MOBILE SLIDESHOW FAQs -->
            <div class="relative md:hidden mt-4">
              <!-- Flecha izquierda -->
              <button
                type="button"
                on:click={prevFaq}
                class="absolute left-2 top-1/2 -translate-y-1/2 z-10 text-white disabled:opacity-30 disabled:cursor-default"
                aria-label="FAQ anterior"
                disabled={currentFaqIndex === 0}
              >
                <svg class="w-7 h-7" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                </svg>
              </button>

              <!-- Flecha derecha -->
              <button
                type="button"
                on:click={nextFaq}
                class="absolute right-2 top-1/2 -translate-y-1/2 z-10 text-white disabled:opacity-30 disabled:cursor-default"
                aria-label="Siguiente FAQ"
                disabled={currentFaqIndex === totalFaqs - 1}
              >
                <svg class="w-7 h-7" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                </svg>
              </button>

              <!-- Contenedor de slide -->
              <div class="overflow-hidden">
                <div
                  class="flex transition-transform duration-300"
                  style={`transform: translateX(-${currentFaqIndex * 100}%);`}
                >
                  <!-- FAQ 1 -->
                  <div class="w-full flex justify-center px-10 flex-shrink-0">
                    <div class="w-full max-w-xs bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                        <h4 class="font-bold text-white mb-2 text-lg">¿Cuánto tardáis en entregar?</h4>
                        <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                            Entregamos webs corporativas estándar en 2 a 4 semanas tras recibir todo el material. Proyectos complejos se planifican con un plazo ajustado a cada caso.
                        </p>
                    </div>
                  </div>

                  <!-- FAQ 2 -->
                  <div class="w-full flex justify-center px-10 flex-shrink-0">
                    <div class="w-full max-w-xs bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                        <h4 class="font-bold text-white mb-2 text-lg">¿Qué incluye el servicio?</h4>
                        <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                            Incluye diseño y desarrollo a medida, dominio, hosting optimizado, mantenimiento técnico y la entrega total de la propiedad intelectual del proyecto.
                        </p>
                    </div>
                  </div>

                  <!-- FAQ 3 -->
                  <div class="w-full flex justify-center px-10 flex-shrink-0">
                    <div class="w-full max-w-xs bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                        <h4 class="font-bold text-white mb-2 text-lg">¿Por qué elegir Poldan?</h4>
                        <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                            Diseñamos y construimos activos digitales, mejorando tu presencia online con SEO, arquitectura moderna y experiencia de usuario optimizada para tus objetivos.
                        </p>
                    </div>
                  </div>

                  <!-- FAQ 4 -->
                  <div class="w-full flex justify-center px-10 flex-shrink-0">
                    <div class="w-full max-w-xs bg-white/5 p-6 rounded-2xl border border-white/5 h-full flex flex-col">
                        <h4 class="font-bold text-white mb-2 text-lg">¿Tengo que pagar todo ya?</h4>
                        <p class="text-gray-400 text-sm leading-relaxed text-justify flex-grow">
                            No. Trabajamos con 50% al inicio y 50% al finalizar. Así aseguramos seguridad y que estés 100% satisfecho con el resultado antes de pagar lo restante.
                        </p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
        </section>

        <!-- CONTACT SECTION -->
        <section id="contact" class="pt-25 pb-28 px-6 border-t border-white/5 bg-gradient-to-b from-[#030c09] to-[#05100c] scroll-mt-19">
          <div class="max-w-2xl mx-auto text-center w-full">
            <h2 class="text-3xl md:text-4xl font-bold mb-8 text-center">Empieza tu proyecto</h2>
            <p class="text-gray-400 mb-10">¿Tienes una idea o necesitas mejorar tu presencia actual? Hablemos sin compromiso.</p>

            <form on:submit|preventDefault={handleSubmit} class="space-y-4 text-left relative">
              <!-- Checkbox anti-spam oculto -->
              <input type="checkbox" name="botcheck" class="hidden" style="display: none;">

              <div>
                <label for="email" class="block text-xs uppercase tracking-wider text-gray-500 mb-2 ml-1">Correo electrónico</label>
                <input 
                  type="email" 
                  name="email" 
                  id="email" 
                  required 
                  placeholder="tu@email.com" 
                  class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#51ad7c] focus:ring-1 focus:ring-[#51ad7c] transition-all placeholder-gray-600" 
                />
              </div>

              <div>
                <label for="message" class="block text-xs uppercase tracking-wider text-gray-500 mb-2 ml-1">Mensaje</label>
                <textarea 
                  name="message" 
                  id="message" 
                  rows="4" 
                  required 
                  placeholder="Cuéntanos sobre tu proyecto..." 
                  class="w-full bg-white/5 border border-white/10 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#51ad7c] focus:ring-1 focus:ring-[#51ad7c] transition-all placeholder-gray-600"
                ></textarea>
              </div>

              <button 
                type="submit" 
                disabled={formStatus === 'submitting'}
                class="w-full bg-gradient-to-r from-[#51ad7c] to-emerald-700 hover:brightness-110 text-white font-bold py-4 rounded-xl transition-all shadow-lg transform active:scale-95 mt-4 border border-[#51ad7c]/20 flex justify-center items-center gap-2 disabled:opacity-70 disabled:cursor-wait"
              >
                {#if formStatus === 'submitting'}
                  <svg class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                  </svg>
                  <span>Enviando...</span>
                {:else}
                  Enviar Mensaje
                {/if}
              </button>
            </form>
          </div>
        </section>
      </div>
    </div>
  {/if}

  <!-- CONTENIDO ABOUT -->
  {#if currentPath === 'about'}
    <!-- Aplicamos transición fade rápida para que parezca un corte suave -->
    <div in:fade={{ duration: 300 }} class="pt-32 min-h-screen"> 
      
      <!-- BOTÓN VOLVER (Solo Flecha) -->
      <div class="max-w-7xl mx-auto px-6 mb-4">
        <button 
          on:click={() => navigateTo('home')} 
          class="group flex items-center p-2 -ml-2 text-gray-400 hover:text-[#51ad7c] transition-colors"
          aria-label="Volver"
        >
          <!-- Icono Chevron Left simple (<) -->
          <!-- La clase group-hover:-translate-x-2 hace la animación de moverse a la izquierda -->
          <svg class="w-8 h-8 group-hover:-translate-x-2 transition-transform duration-300" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
          </svg>
        </button>
      </div>

      <section id="about" class="pb-12 px-6 max-w-7xl mx-auto scroll-mt-20">
        <div class="text-center mb-16">
          <h2 class="text-3xl md:text-4xl font-bold mb-6 text-center">Nosotros</h2>
          <p class="text-xl text-gray-400 max-w-3xl mx-auto">
            Alianza estratégica entre España y EE. UU. para crear webs que venden y comunican.
          </p>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-12 items-stretch">
          <!-- Texto -->
          <div class="space-y-8 flex flex-col justify-center">
            <div>
                <h3 class="text-2xl font-bold text-white mb-3">Tecnología y Negocio</h3>
                <p class="text-gray-400 leading-relaxed text-justify">
                  Somos una agencia formada por dos perfiles complementarios: desarrollo técnico de alto nivel y estrategia empresarial enfocada a resultados. Trabajamos entre España y EE.UU., aportando una perspectiva global a negocios locales.
                </p>
            </div>
            
            <div class="space-y-4">
              <!-- Perfil 1 -->
              <div class="bg-white/5 p-6 rounded-2xl border border-white/5 hover:border-[#51ad7c]/30 transition-colors">
                <div class="font-bold text-white text-lg">Ingeniería y Desarrollo</div>
                <div class="text-xs text-[#51ad7c] tracking-wider mb-2 uppercase font-semibold">Daniel</div>
                <p class="text-sm text-gray-400 text-justify">
                  Graduado en Ingeniería Informática por la <strong>Universidad Autónoma de Barcelona</strong> y la <strong>University of Montana</strong>. Transforma ideas en webs rápidas, modernas y optimizadas para convertir visitas en clientes.
                </p>
              </div>

              <!-- Perfil 2 -->
              <div class="bg-white/5 p-6 rounded-2xl border border-white/5 hover:border-[#51ad7c]/30 transition-colors">
                <div class="font-bold text-white text-lg">Estrategia y Negocio</div>
                <div class="text-xs text-[#51ad7c] tracking-wider mb-2 uppercase font-semibold">Pol</div>
                <p class="text-sm text-gray-400 text-justify">
                  Estudiante de ADE en la <strong>Universidad Autónoma de Barcelona</strong>. Enfocado en crear páginas web orientadas a la rentabilidad, con estructuras claras y mensajes que conectan con tu público.
                </p>
              </div>
            </div>
          </div>

          <!-- Mapa Global -->
          <div class="bg-[#05100c] rounded-3xl border border-white/10 relative overflow-hidden group h-full min-h-[400px]">
              <img
                src="/world-map-dots.png" 
                class="absolute top-0 left-0 w-full h-full object-cover opacity-40 mix-blend-screen pointer-events-none z-0"
                alt="World Map"
              />
              <div class="absolute inset-0 w-full h-full z-10">
                  <svg viewBox="0 0 1000 500" preserveAspectRatio="xMidYMid slice" fill="none" xmlns="http://www.w3.org/2000/svg" class="w-full h-full">
                      <path d="M-50,50 Q150,20 300,80 T500,200 T400,450 T150,500 T-50,300 Z" fill="#51ad7c" fill-opacity="0.01" stroke="#51ad7c" stroke-opacity="0.1" />
                      <path d="M600,50 Q800,20 950,100 T1050,400 T750,500 T550,300 Z" fill="#51ad7c" fill-opacity="0.01" stroke="#51ad7c" stroke-opacity="0.1" />
                      <path 
                        d="M405,208 Q505,120 605,213" 
                        stroke="#51ad7c" 
                        stroke-width="2" 
                        stroke-dasharray="6 6" 
                        stroke-opacity="0.6" 
                        fill="none"
                        class="animate-dash" 
                      />
                      <!-- Markers -->
                      <foreignObject x="305" y="200" width="200" height="200" style="overflow: visible; z-index: 50;">
                        <div xmlns="http://www.w3.org/1999/xhtml" class="flex flex-col items-center group/marker relative z-50">
                          <div class="relative">
                            <div class="w-4 h-4 bg-[#51ad7c] rounded-full animate-pulse shadow-[0_0_15px_rgba(81,173,124,0.8)]"></div>
                            <div class="absolute -inset-4 border border-[#51ad7c]/30 rounded-full scale-0 group-hover/marker:scale-100 transition-transform duration-500"></div>
                          </div>
                          <div class="mt-3 text-center backdrop-blur-md bg-black/80 px-4 py-1.5 rounded-xl border border-white/20 shadow-2xl z-50 relative">
                            <div class="font-bold text-white text-sm shadow-black drop-shadow-md">Montana</div>
                            <div class="text-[10px] text-gray-300 uppercase tracking-wider font-semibold">EE. UU.</div>
                          </div>
                        </div>
                      </foreignObject>
                      <foreignObject x="505" y="205" width="200" height="200" style="overflow: visible; z-index: 50;">
                        <div xmlns="http://www.w3.org/1999/xhtml" class="flex flex-col items-center group/marker relative z-50">
                          <div class="relative">
                            <div class="w-4 h-4 bg-[#51ad7c] rounded-full animate-pulse delay-700 shadow-[0_0_15px_rgba(81,173,124,0.8)]"></div>
                            <div class="absolute -inset-4 border border-[#51ad7c]/30 rounded-full scale-0 group-hover/marker:scale-100 transition-transform duration-500"></div>
                          </div>
                          <div class="mt-3 text-center backdrop-blur-md bg-black/80 px-4 py-1.5 rounded-xl border border-white/20 shadow-2xl z-50 relative">
                            <div class="font-bold text-white text-sm shadow-black drop-shadow-md">Barcelona</div>
                            <div class="text-[10px] text-gray-300 uppercase tracking-wider font-semibold">España</div>
                          </div>
                        </div>
                      </foreignObject>
                  </svg>
              </div>
              <div class="absolute bottom-8 left-0 right-0 text-center px-4 z-20 pointer-events-none">
                  <p class="text-sm text-gray-300 max-w-md mx-auto backdrop-blur-md py-3 px-6 rounded-full bg-black/40 border border-white/10 shadow-lg">
                      Conocimiento técnico y empresarial para tu negocio.
                  </p>
              </div>
          </div>
        </div>
      </section>
    </div>
  {/if}

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

  <!-- Notificación Flotante de Éxito -->
  {#if showNotification}
    <div 
      in:fly={{ y: -50, duration: 500 }} 
      out:fade 
      class="fixed top-24 left-1/2 -translate-x-1/2 z-50 w-[90%] md:w-full max-w-sm bg-[#05100c] border border-[#51ad7c] shadow-[0_0_30px_rgba(81,173,124,0.3)] rounded-2xl p-4 flex items-center gap-4"
    >
      <!-- Icono Check -->
      <div class="bg-[#51ad7c]/20 p-2 rounded-full flex-shrink-0">
        <svg class="w-6 h-6 text-[#51ad7c]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
        </svg>
      </div>
      
      <!-- Texto -->
      <div>
        <h4 class="text-white font-bold text-sm">¡Mensaje Enviado!</h4>
        <p class="text-gray-400 text-xs mt-1 leading-relaxed">
          Hemos recibido tu correo.
          <span class="block mt-0.5">Te contactaremos en menos de 48 horas.</span>
        </p>
      </div>

      <!-- Botón Cerrar -->
      <button type="button" aria-label="Cerrar notificación" on:click={() => showNotification = false} class="ml-auto text-gray-500 hover:text-white transition-colors">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
        </svg>
      </button>
    </div>
  {/if}
  
</div>

<style>
  :global(html) {
    scroll-behavior: smooth;
  }

  @keyframes pulse-slow {
    0%, 100% { opacity: 0.10; transform: translate(-50%, -50%) scale(1); }
    50% { opacity: 0.15; transform: translate(-50%, -50%) scale(1.1); }
  }
  .animate-pulse-slow { animation: pulse-slow 8s ease-in-out infinite; }

  @keyframes dash {
    to {
      stroke-dashoffset: -12; 
    }
  }
  
  .animate-dash {
    animation: dash 1s linear infinite;
  }

  /* Animación de scaneo para la línea de proceso */
  @keyframes scan {
    0% {
        left: -20%;
    }
    100% {
        left: 120%;
    }
  }
  .animate-scan {
    animation: scan 3s linear infinite;
  }
</style>

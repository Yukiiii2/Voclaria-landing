<script setup>
import { ref, onMounted } from 'vue'
import QRCode from 'qrcode'

// Direct file in /public (no code changes needed for hosting/CDN later)
const DOWNLOAD_URL = '/Voclaria(Final Version).apk'

// import the image from src/assets (works with Vite alias @ -> /src)
import logoUrl from '@/assets/Speaksy.png' // or: '../assets/Speaksy.png'

// QR data URL (desktop only)
const qrDataUrl = ref('')

// Generate QR that points to the *absolute* APK URL so it works when scanned on a phone
onMounted(async () => {
  const absoluteApkUrl = new URL(DOWNLOAD_URL, window.location.origin).toString()
  try {
    qrDataUrl.value = await QRCode.toDataURL(absoluteApkUrl, {
      errorCorrectionLevel: 'M',
      margin: 1,
      width: 720, // plenty of pixels; it will be sized by CSS
      color: { dark: '#ffffff', light: '#00000000' } // white modules on transparent
    })
  } catch (e) {
    console.error('Failed generating QR:', e)
  }
})
</script>

<template>
  <section
    class="relative min-h-screen overflow-hidden text-white bg-gradient-to-b from-[#0F172A] via-[#1E293B] to-[#0F172A]"
  >
    <!-- Decorative blobs -->
    <div class="pointer-events-none">
      <div class="absolute w-40 h-40 bg-purple-400/10 rounded-full -top-28 -left-12" />
      <div class="absolute w-24 h-24 bg-purple-400/10 rounded-full top-36 left-64 hidden xl:block" />
      <div class="absolute w-36 h-36 bg-purple-400/10 rounded-full -bottom-10 -right-8" />
    </div>

    <!-- STICKY NAV (unchanged desktop look) -->
    <header
      class="sticky top-0 z-50 border-b border-white/10 bg-[#0F172A]/70 backdrop-blur supports-[backdrop-filter]:bg-[#0F172A]/50"
    >
      <div class="w-full max-w-[120rem] mx-auto px-5 sm:px-6 lg:px-10">
        <div class="h-16 flex items-center justify-between">
          <div class="flex items-center gap-2">
            <img :src="logoUrl" alt="Voclaria" class="w-10 h-10 -ml-1" />
            <h1 class="font-extrabold tracking-tight text-[clamp(1.5rem,2.2vw,2.5rem)]">
              Voclaria
            </h1>
          </div>

          <nav class="hidden md:block">
            <div class="ml-10 flex items-baseline space-x-8">
              <a href="#features" class="text-gray-300 hover:text-white px-3 py-2 text-sm font-medium transition-colors">Features</a>
              <a href="#contact" class="text-gray-300 hover:text-white px-3 py-2 text-sm font-medium transition-colors">Contact</a>
            </div>
          </nav>
        </div>
      </div>
    </header>

    <!-- HERO -->
    <div class="relative z-10">
      <div class="w-full max-w-[120rem] mx-auto px-5 sm:px-6 lg:px-10">
        <!-- tighter mobile padding; desktop unchanged -->
        <div class="py-8 sm:py-10 md:py-12 lg:py-16">
          <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 md:gap-10 xl:gap-14 items-center">
            <!-- Left column: text (7 cols) -->
            <div class="lg:col-span-7">
              <div class="space-y-1.5 sm:space-y-2 mb-4">
                <!-- mobile-down sizes reduced; lg stays large -->
                <h1 class="font-extrabold leading-[1.05] text-[1.75rem] sm:text-[2.25rem] md:text-[2.75rem] lg:text-[3.75rem]">
                  Voclaria:
                </h1>
                <h1 class="font-extrabold leading-[1.05] text-[1.75rem] sm:text-[2.25rem] md:text-[2.75rem] lg:text-[3.75rem]">
                  AI Simulation for
                </h1>
                <h1 class="font-extrabold leading-[1.05] text-purple-400 text-[1.75rem] sm:text-[2.25rem] md:text-[2.75rem] lg:text-[3.75rem]">
                  Communication & English Literacy Skills
                </h1>
              </div>

              <!-- smaller paragraph + tighter leading on mobile -->
              <p class="text-gray-200 text-[0.9rem] sm:text-[0.95rem] md:text-[1rem] leading-7 sm:leading-8 max-w-[60ch] md:max-w-[65ch] mb-6 md:mb-7">
                Voclaria is an AI-powered mobile app for Senior High School learners that builds
                speaking confidence and reading fluency through real-time, personalized feedback.
                Students complete curriculum-aligned speaking and reading tasks, then receive instant
                insights on pronunciation, fluency, pacing, clarity, and grammar. Teachers can create
                classes, upload lesson materials, and monitor progress in a dedicated dashboard, while
                strand- and grade-based access ensures learners see the right modules. Designed with
                privacy and responsible use in mind, Voclaria helps learners practice meaningfully and
                improve communication performance on Android devices.
              </p>

              <!-- MOBILE-ONLY download button (QR hidden on mobile) -->
              <div class="lg:hidden">
                <a
                  :href="DOWNLOAD_URL"
                  class="inline-flex items-center justify-center bg-purple-500 hover:bg-purple-500/90 transition-colors border border-white/30 px-5 py-3 rounded-lg text-[0.9rem] font-semibold shadow-lg w-full sm:w-auto"
                >
                  Download for Android
                </a>
              </div>

              <!-- Social proof -->
              <div class="flex items-center mt-4">
                <div class="flex -space-x-2">
                  <div class="w-8 h-8 rounded-full bg-white/10 grid place-items-center">
                    <span class="text-white/80 text-sm font-bold">X</span>
                  </div>
                  <div class="w-8 h-8 rounded-full bg-white/10 grid place-items-center">
                    <span class="text-white/80 text-sm font-bold">E</span>
                  </div>
                  <div class="w-8 h-8 rounded-full bg-white/10 grid place-items-center">
                    <span class="text-white/80 text-sm font-bold">Y</span>
                  </div>
                  <div class="w-8 h-8 rounded-full bg-white/10 grid place-items-center">
                    <span class="text-white/80 text-sm font-bold">F</span>
                  </div>
                </div>
                <p class="text-gray-400 text-xs ml-3">
                  <span class="text-white font-medium">10,000+</span> Active learners
                </p>
              </div>
            </div>

            <!-- Right column (5 cols):
                 DESKTOP: QR code card (replaces mockup)
                 MOBILE: hidden -->
            <div class="lg:col-span-5 hidden lg:block">
              <div
                class="rounded-2xl border border-white/20 bg-white/5 backdrop-blur-sm w-full grid place-items-center"
                :class="'h-[clamp(22rem,34vw,40rem)]'"
              >
                <div class="flex flex-col items-center">
                  <img
                    v-if="qrDataUrl"
                    :src="qrDataUrl"
                    alt="Scan to download the Android APK"
                    class="w-[min(22rem,60%)] h-auto rounded-xl shadow-[0_8px_32px_rgba(0,0,0,0.35)]"
                  />
                  <div class="mt-4 text-center text-sm text-gray-300">
                    Scan with your phone to download the Android APK
                  </div>
                  <noscript>
                    <div class="mt-3">
                      <a :href="DOWNLOAD_URL" class="underline text-gray-200">Download APK</a>
                    </div>
                  </noscript>
                </div>
              </div>
            </div>
            <!-- End right column -->
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

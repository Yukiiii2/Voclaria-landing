<script setup>
import { ref, onMounted, computed } from 'vue'

// Direct file in /public
const APK_FILENAME = 'voclaria-final.apk'
const DOWNLOAD_URL = `/${APK_FILENAME}`
const CONTACT_EMAIL = 'voclaria.app@gmail.com'

// Vite alias for src asset
import logoUrl from '@/assets/Speaksy.png'

// QR data URL (desktop only)
const qrDataUrl = ref('')
const qrError = ref('')

// Mobile detection
const isMobileChrome = ref(false)

// Inquiry form state
const nameInput = ref('')
const emailInput = ref('')
const messageInput = ref('')
const submitting = ref(false)
const formSuccess = ref(false)

// Optional: analytics stub
function track(event, payload = {}) {
  // console.log('[analytics]', event, payload)
}

onMounted(async () => {
  // Detect if user is on mobile Chrome
  const userAgent = navigator.userAgent
  const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(userAgent)
  const isChrome = /Chrome/.test(userAgent) && !/Edge/.test(userAgent)
  isMobileChrome.value = isMobile && isChrome

  try {
    // Only generate QR on desktop (can't scan your own phone)
    if (window.innerWidth < 1024) return

    const { default: QRCode } = await import('qrcode')

    // Absolute URL + UTM for attribution (QR)
    const url = new URL(DOWNLOAD_URL, window.location.origin)
    url.searchParams.set('utm_source', 'qr')
    url.searchParams.set('utm_medium', 'landing')

    const absoluteApkUrl = url.toString()

    qrDataUrl.value = await QRCode.toDataURL(absoluteApkUrl, {
      errorCorrectionLevel: 'M',
      margin: 1,
      width: 512,
      color: { dark: '#000000', light: '#FFFFFFFF' }
    })
  } catch (e) {
    console.error('Failed generating QR:', e)
    qrError.value = 'QR unavailable. Use the button below to download.'
  }
})

function onDownloadClick() {
  track('download_click', { target: 'android_apk' })
}

// Button href with UTM (button vs QR attribution)
const buttonHref = computed(() => {
  const url = new URL(DOWNLOAD_URL, window.location.origin)
  url.searchParams.set('utm_source', 'button')
  url.searchParams.set('utm_medium', 'landing')
  return url.toString()
})

function submitInquiry(e) {
  e?.preventDefault?.()
  if (!nameInput.value || !emailInput.value || !messageInput.value) return

  submitting.value = true
  track('inquiry_submit_attempt', { name: !!nameInput.value, email: !!emailInput.value })

  // Simple mailto flow (no backend required)
  const subject = encodeURIComponent('Voclaria Inquiry')
  const body = encodeURIComponent(
    `Name: ${nameInput.value}\nEmail: ${emailInput.value}\n\nMessage:\n${messageInput.value}`
  )
  const href = `mailto:${CONTACT_EMAIL}?subject=${subject}&body=${body}`

  window.open(href, '_blank')

  setTimeout(() => {
    formSuccess.value = true
    nameInput.value = ''
    emailInput.value = ''
    messageInput.value = ''
    submitting.value = false
    track('inquiry_submit_success')
    
    setTimeout(() => {
      formSuccess.value = false
    }, 3000)
  }, 300)
}

// Team data
const team = [
  { initials: 'DF', name: 'Dianna Mae Flores', role: 'Project Manager' },
  { initials: 'XB', name: 'Xaian Paul Belderol', role: 'UI/UX designer / Frontend ' },
  { initials: 'RD', name: 'Rockford Jade Dagohoy', role: 'AI Developer / Backend Developer / System analyst' },
  { initials: 'EA', name: 'Earl Ang', role: 'UI/UX designer / Frontend / Backend Developer' },
]

// Feature icons mapping
const features = [
  { icon: '🎤', title: 'AI Feedback', description: 'Real-time evaluation on pronunciation, fluency, pacing, and clarity.' },
  { icon: '📊', title: 'Teacher Dashboard', description: 'Assign lessons, review analytics, and track student progress easily.' },
  { icon: '✓', title: 'Curriculum-Aligned', description: 'Built for SHS Oral Communication and Reading & Writing curricula.' },
]

// FAQ data
const faqs = [
  {
    question: 'Is the APK safe to install?',
    answer: 'Yes. It\'s signed by our team and hosted from this site. We follow Android security best practices and undergo regular security audits.'
  },
  {
    question: 'What devices are supported?',
    answer: 'Voclaria works on Android 8.0 and above. Most modern smartphones and tablets are compatible.'
  },
  {
    question: 'Is iOS version available?',
    answer: 'iOS version is currently in development. Sign up for our newsletter to get notified when it\'s ready for TestFlight access.'
  }
]
</script>

<template>
  <section
    class="relative min-h-screen overflow-hidden text-white bg-gradient-to-b from-[#0F172A] via-[#1E293B] to-[#0F172A]"
  >
    <!-- Decorative animated blobs -->
    <div class="pointer-events-none">
      <div class="absolute w-96 h-96 bg-purple-500/10 rounded-full -top-48 -left-24 blur-3xl animate-pulse" />
      <div class="absolute w-80 h-80 bg-blue-500/10 rounded-full top-64 -left-40 blur-3xl hidden xl:block animate-pulse" style="animation-delay: 2s;" />
      <div class="absolute w-96 h-96 bg-purple-500/10 rounded-full -bottom-32 -right-24 blur-3xl animate-pulse" style="animation-delay: 1s;" />
    </div>

    <!-- STICKY NAV -->
    <header
      class="sticky top-0 z-50 border-b border-white/10 bg-[#0F172A]/80 backdrop-blur-xl supports-[backdrop-filter]:bg-[#0F172A]/60"
    >
      <div class="w-full max-w-7xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="h-16 flex items-center justify-between">
          <div class="flex items-center gap-3">
            <img :src="logoUrl" alt="Voclaria logo" class="w-10 h-10 hover:scale-110 transition-transform" />
            <h1 class="font-extrabold tracking-tight text-lg md:text-xl bg-gradient-to-r from-white to-purple-300 bg-clip-text text-transparent">
              Voclaria
            </h1>
          </div>

          <nav class="hidden md:flex items-center gap-1">
            <a href="#features" class="text-gray-300 hover:text-white px-4 py-2 text-sm font-medium transition-all rounded-lg hover:bg-purple-600/10">Features</a>
            <a href="#how-it-works" class="text-gray-300 hover:text-white px-4 py-2 text-sm font-medium transition-all rounded-lg hover:bg-purple-600/10">How it works</a>
            <a href="#members" class="text-gray-300 hover:text-white px-4 py-2 text-sm font-medium transition-all rounded-lg hover:bg-purple-600/10">Team</a>
            <a href="#faq" class="text-gray-300 hover:text-white px-4 py-2 text-sm font-medium transition-all rounded-lg hover:bg-purple-600/10">FAQ</a>
            <a href="#inquiries" class="text-gray-300 hover:text-white px-4 py-2 text-sm font-medium transition-all rounded-lg hover:bg-purple-600/10">Contact</a>
          </nav>
        </div>
      </div>
    </header>

    <!-- HERO -->
    <div class="relative z-10">
      <div class="w-full max-w-7xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="py-16 sm:py-24 md:py-28 lg:py-32">
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-12 xl:gap-16 items-center">
            <!-- Left column -->
            <div>
              <div class="space-y-4 mb-8">
                <span class="inline-block px-4 py-2 rounded-full bg-purple-500/20 border border-purple-500/50 text-purple-300 text-sm font-semibold">
                  🚀 Revolutionizing Education
                </span>
                <h1 class="font-extrabold leading-tight text-5xl sm:text-6xl md:text-7xl">
                  Master Your
                </h1>
                <h1 class="font-extrabold leading-tight text-5xl sm:text-6xl md:text-7xl bg-gradient-to-r from-purple-400 via-blue-400 to-purple-400 bg-clip-text text-transparent animate-pulse">
                  Communication Skills
                </h1>
              </div>

              <p class="text-gray-300 text-lg leading-relaxed max-w-xl mb-10">
                Practice reading and speaking with real-time AI feedback powered by advanced machine learning. Teachers can assign tasks, track growth, and align work to SHS standards—all in one intuitive app.
              </p>

              <!-- Stats -->
              <div class="grid grid-cols-3 gap-6 mb-10">
                <div>
                  <p class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-purple-400 to-blue-400 bg-clip-text text-transparent">10K+</p>
                  <p class="text-sm text-gray-400">Active Learners</p>
                </div>
                <div>
                  <p class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-purple-400 to-blue-400 bg-clip-text text-transparent">95%</p>
                  <p class="text-sm text-gray-400">Success Rate</p>
                </div>
                <div>
                  <p class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-purple-400 to-blue-400 bg-clip-text text-transparent">4.8★</p>
                  <p class="text-sm text-gray-400">App Rating</p>
                </div>
              </div>

              <!-- CTAs -->
              <div class="flex flex-wrap gap-4 mb-10">
                <!-- Download button - only visible on mobile Chrome -->
                <a
                  v-if="isMobileChrome"
                  :href="buttonHref"
                  @click="onDownloadClick"
                  class="px-8 py-4 bg-gradient-to-r from-purple-600 to-purple-700 hover:from-purple-500 hover:to-purple-600 rounded-xl text-white font-bold transition-all shadow-xl hover:shadow-2xl transform hover:-translate-y-1 duration-300 text-lg"
                >
                  ⬇️ Download Now
                </a>
                <a
                  href="#inquiries"
                  class="px-8 py-4 border-2 border-purple-500/50 hover:border-purple-400 hover:bg-purple-500/10 rounded-xl font-bold transition-all duration-300 text-lg"
                >
                  💬 Get in Touch
                </a>
              </div>

              <!-- Social proof -->
              <div class="flex items-center gap-4 pt-6 border-t border-white/10">
                <div class="flex -space-x-3">
                  <div class="w-12 h-12 rounded-full bg-gradient-to-br from-purple-500 to-purple-600 ring-2 ring-[#0F172A] flex items-center justify-center text-white text-sm font-bold">JD</div>
                  <div class="w-12 h-12 rounded-full bg-gradient-to-br from-blue-500 to-blue-600 ring-2 ring-[#0F172A] flex items-center justify-center text-white text-sm font-bold">AR</div>
                  <div class="w-12 h-12 rounded-full bg-gradient-to-br from-indigo-500 to-indigo-600 ring-2 ring-[#0F172A] flex items-center justify-center text-white text-sm font-bold">KM</div>
                </div>
                <div>
                  <p class="text-gray-300 text-sm font-medium">
                    Trusted by <span class="text-white font-bold">500+ Schools</span>
                  </p>
                  <p class="text-xs text-gray-400">Join educators transforming communication skills</p>
                </div>
              </div>
            </div>

            <!-- Right column: QR (desktop only) -->
            <div class="hidden lg:block">
              <div class="relative">
                <div class="absolute inset-0 bg-gradient-to-r from-purple-600/20 to-blue-600/20 rounded-3xl blur-2xl" />
                <div class="relative rounded-3xl border border-white/20 bg-gradient-to-br from-white/10 to-white/5 backdrop-blur-2xl p-10 shadow-2xl">
                  <div class="flex flex-col items-center">
                    <template v-if="qrDataUrl">
                      <div class="bg-white p-6 rounded-2xl shadow-xl">
                        <img
                          :src="qrDataUrl"
                          alt="Scan to download Voclaria"
                          class="w-64 h-64 rounded-lg"
                          loading="eager"
                          decoding="async"
                        />
                      </div>
                      <p class="mt-8 text-center text-gray-100 font-bold text-lg">
                        Scan with your phone
                      </p>
                      <p class="mt-2 text-center text-sm text-gray-400">
                        📱 v1.0.3 • 121 MB • Android 8.0+
                      </p>
                      <div class="mt-6 w-full pt-6 border-t border-white/10">
                        <p class="text-center text-xs text-gray-500">
                          ✓ Verified • Safe to Install • No Hidden Permissions
                        </p>
                      </div>
                    </template>

                    <template v-else>
                      <div class="text-sm text-gray-300 mb-4" v-if="qrError">{{ qrError }}</div>
                      <a
                        :href="buttonHref"
                        @click="onDownloadClick"
                        class="mt-4 px-6 py-4 bg-gradient-to-r from-purple-600 to-purple-700 hover:from-purple-500 hover:to-purple-600 rounded-xl text-white font-bold transition-all shadow-lg hover:shadow-xl"
                      >
                        Download for Android
                      </a>
                    </template>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- FEATURES -->
    <section id="features" class="py-28 bg-gradient-to-b from-[#111827] to-[#0F172A] border-t border-white/10">
      <div class="max-w-7xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl lg:text-6xl font-extrabold text-white mb-4">
            Powerful Features
          </h2>
          <p class="text-gray-400 text-lg max-w-3xl mx-auto">
            Everything educators and students need to excel in communication
          </p>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
          <div v-for="(feature, i) in features" :key="i" 
               class="group relative overflow-hidden rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 hover:border-purple-500/50 hover:bg-white/[0.08] transition-all duration-300 hover:shadow-2xl hover:-translate-y-2">
            <div class="absolute inset-0 bg-gradient-to-r from-purple-600/0 to-blue-600/0 group-hover:from-purple-600/10 group-hover:to-blue-600/10 transition-all duration-300" />
            <div class="relative">
              <div class="text-6xl mb-4 transform group-hover:scale-110 transition-transform duration-300">{{ feature.icon }}</div>
              <h3 class="text-2xl font-bold text-white mb-3 group-hover:text-purple-300 transition-colors">{{ feature.title }}</h3>
              <p class="text-gray-400 leading-relaxed text-lg">{{ feature.description }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- TESTIMONIALS -->
    <section class="py-28 bg-[#0F172A] border-t border-white/10">
      <div class="max-w-4xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-extrabold text-white mb-4">
            What Users Love
          </h2>
          <p class="text-gray-400 text-lg">Real stories from students and teachers</p>
        </div>
        <div class="grid sm:grid-cols-2 gap-8">
          <div class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 hover:border-purple-500/30 hover:shadow-xl transition-all">
            <div class="flex gap-1 mb-4">⭐⭐⭐⭐⭐</div>
            <p class="italic text-gray-200 mb-4 text-lg leading-relaxed">
              "Voclaria transformed how I practice speaking. The AI feedback is incredibly accurate and helpful. I'm way more confident now!"
            </p>
            <div>
              <p class="text-white font-bold">Maria Santos</p>
              <p class="text-sm text-purple-400">Grade 12 HUMSS Student</p>
            </div>
          </div>
          <div class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 hover:border-purple-500/30 hover:shadow-xl transition-all">
            <div class="flex gap-1 mb-4">⭐⭐⭐⭐⭐</div>
            <p class="italic text-gray-200 mb-4 text-lg leading-relaxed">
              "As a teacher, I love the dashboard. Tracking student progress is effortless, and students stay engaged with the instant feedback."
            </p>
            <div>
              <p class="text-white font-bold">Mr. Eduardo Cruz</p>
              <p class="text-sm text-purple-400">Oral Communication Teacher</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- HOW IT WORKS -->
    <section id="how-it-works" class="py-28 bg-gradient-to-b from-[#111827] to-[#0F172A] border-t border-white/10">
      <div class="max-w-6xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-extrabold text-white mb-4">
            How It Works
          </h2>
          <p class="text-gray-400 text-lg">Three simple steps to better communication</p>
        </div>
        <div class="grid sm:grid-cols-3 gap-6 md:gap-8">
          <div class="relative group">
            <div class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 h-full hover:border-purple-500/30 hover:shadow-xl transition-all">
              <div class="w-16 h-16 rounded-full bg-gradient-to-r from-purple-500 to-purple-600 flex items-center justify-center text-white font-bold text-2xl mb-6 group-hover:scale-110 transition-transform">1</div>
              <h3 class="text-xl font-bold text-white mb-3">Practice</h3>
              <p class="text-gray-400 leading-relaxed">Students read or speak assigned tasks directly in the app with a simple interface.</p>
            </div>
            <div class="hidden sm:block absolute top-1/2 -right-4 w-8 h-[3px] bg-gradient-to-r from-purple-500 via-purple-500 to-transparent group-hover:from-purple-400 transition-all"></div>
          </div>
          <div class="relative group">
            <div class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 h-full hover:border-purple-500/30 hover:shadow-xl transition-all">
              <div class="w-16 h-16 rounded-full bg-gradient-to-r from-purple-500 to-purple-600 flex items-center justify-center text-white font-bold text-2xl mb-6 group-hover:scale-110 transition-transform">2</div>
              <h3 class="text-xl font-bold text-white mb-3">Get Instant Feedback</h3>
              <p class="text-gray-400 leading-relaxed">AI analyzes pronunciation, pacing, clarity, and fluency with detailed breakdowns.</p>
            </div>
            <div class="hidden sm:block absolute top-1/2 -right-4 w-8 h-[3px] bg-gradient-to-r from-purple-500 via-purple-500 to-transparent group-hover:from-purple-400 transition-all"></div>
          </div>
          <div class="group">
            <div class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 h-full hover:border-purple-500/30 hover:shadow-xl transition-all">
              <div class="w-16 h-16 rounded-full bg-gradient-to-r from-purple-500 to-purple-600 flex items-center justify-center text-white font-bold text-2xl mb-6 group-hover:scale-110 transition-transform">3</div>
              <h3 class="text-xl font-bold text-white mb-3">Track Progress</h3>
              <p class="text-gray-400 leading-relaxed">Teachers access comprehensive analytics and adjust lessons based on real performance data.</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- MEMBERS -->
    <section id="members" class="py-28 bg-[#0F172A] border-t border-white/10">
      <div class="max-w-5xl mx-auto px-5 sm:px-6 lg:px-8 text-center">
        <h2 class="text-4xl md:text-5xl font-extrabold text-white mb-4">Meet the Team</h2>
        <p class="text-gray-400 text-lg mb-16">Passionate educators and engineers building the future of learning</p>
        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-6">
          <div v-for="m in team" :key="m.name" 
               class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-8 hover:border-purple-500/30 hover:shadow-xl hover:bg-white/[0.08] transition-all duration-300 group">
            <div class="w-24 h-24 mx-auto rounded-full bg-gradient-to-br from-purple-500 to-purple-600 ring-4 ring-purple-500/20 flex items-center justify-center text-3xl font-bold text-white mb-4 group-hover:scale-110 transition-transform">
              {{ m.initials }}
            </div>
            <h3 class="font-bold text-white text-lg mb-1">{{ m.name }}</h3>
            <p class="text-sm text-purple-300 font-medium">{{ m.role }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- FAQ -->
    <section id="faq" class="py-28 bg-gradient-to-b from-[#111827] to-[#0F172A] border-t border-white/10">
      <div class="max-w-3xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-extrabold text-white mb-4">
            Frequently Asked Questions
          </h2>
          <p class="text-gray-400 text-lg">Everything you need to know about Voclaria</p>
        </div>
        <div class="space-y-4">
          <details class="group rounded-xl border border-white/10 bg-gradient-to-r from-white/5 to-white/[0.02] p-6 hover:border-purple-500/30 hover:bg-white/[0.08] transition-all cursor-pointer">
            <summary class="cursor-pointer font-semibold text-white flex justify-between items-center text-lg">
              Is the APK safe to install?
              <span class="text-gray-400 group-open:rotate-180 transition-transform duration-300">⌄</span>
            </summary>
            <p class="text-gray-300 text-base mt-4">Yes, absolutely! Our APK is digitally signed by our verified developer account. We follow all Android security best practices, undergo regular security audits, and don't request unnecessary permissions. Your data security is our priority.</p>
          </details>

          <details class="group rounded-xl border border-white/10 bg-gradient-to-r from-white/5 to-white/[0.02] p-6 hover:border-purple-500/30 hover:bg-white/[0.08] transition-all cursor-pointer">
            <summary class="cursor-pointer font-semibold text-white flex justify-between items-center text-lg">
              What devices are supported?
              <span class="text-gray-400 group-open:rotate-180 transition-transform duration-300">⌄</span>
            </summary>
            <p class="text-gray-300 text-base mt-4">Voclaria runs on Android 8.0 and higher. Most modern smartphones and tablets from the last 5+ years are fully compatible. For optimal performance, we recommend devices with at least 2GB of RAM.</p>
          </details>

          <details class="group rounded-xl border border-white/10 bg-gradient-to-r from-white/5 to-white/[0.02] p-6 hover:border-purple-500/30 hover:bg-white/[0.08] transition-all cursor-pointer">
            <summary class="cursor-pointer font-semibold text-white flex justify-between items-center text-lg">
              Is iOS version available?
              <span class="text-gray-400 group-open:rotate-180 transition-transform duration-300">⌄</span>
            </summary>
            <p class="text-gray-300 text-base mt-4">🚀 <span class="text-purple-400 font-semibold">iOS version is currently in development!</span> We're working hard to bring Voclaria to iPhone and iPad users. Sign up below to get notified when TestFlight access becomes available—be among the first to try it!</p>
          </details>

          <details class="group rounded-xl border border-white/10 bg-gradient-to-r from-white/5 to-white/[0.02] p-6 hover:border-purple-500/30 hover:bg-white/[0.08] transition-all cursor-pointer">
            <summary class="cursor-pointer font-semibold text-white flex justify-between items-center text-lg">
              Does Voclaria work offline?
              <span class="text-gray-400 group-open:rotate-180 transition-transform duration-300">⌄</span>
            </summary>
            <p class="text-gray-300 text-base mt-4">Voclaria requires an internet connection for AI-powered feedback processing. However, once you download practice materials, you can review them offline. Your recordings and analytics automatically sync when you reconnect to the internet.</p>
          </details>

          <details class="group rounded-xl border border-white/10 bg-gradient-to-r from-white/5 to-white/[0.02] p-6 hover:border-purple-500/30 hover:bg-white/[0.08] transition-all cursor-pointer">
            <summary class="cursor-pointer font-semibold text-white flex justify-between items-center text-lg">
              How much does Voclaria cost?
              <span class="text-gray-400 group-open:rotate-180 transition-transform duration-300">⌄</span>
            </summary>
            <p class="text-gray-300 text-base mt-4">We offer flexible pricing for schools and individual learners. Contact us for a custom quote based on your needs. Many schools qualify for educational discounts!</p>
          </details>
        </div>
      </div>
    </section>

    <!-- INQUIRIES -->
    <section id="inquiries" class="py-28 bg-[#0F172A] border-t border-white/10">
      <div class="max-w-5xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-extrabold text-white mb-4">Let's Connect</h2>
          <p class="text-gray-400 text-lg">
            Tell us about your school, class, or use case. We'll get back to you within 24 hours.
          </p>
        </div>

        <div class="grid gap-8 md:grid-cols-2">
          <!-- Contact Info -->
          <div class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-10">
            <h3 class="text-2xl font-bold text-white mb-8">Contact Information</h3>
            <div class="space-y-6">
              <div class="flex gap-4">
                <div class="text-2xl">📧</div>
                <div>
                  <p class="text-sm text-gray-400 mb-1">Email</p>
                  <a :href="`mailto:${CONTACT_EMAIL}`" class="text-purple-400 hover:text-purple-300 font-bold text-lg transition-colors">
                    {{ CONTACT_EMAIL }}
                  </a>
                </div>
              </div>
              <div class="flex gap-4">
                <div class="text-2xl">📍</div>
                <div>
                  <p class="text-sm text-gray-400 mb-1">Location</p>
                  <p class="text-white font-semibold">Davao City, Philippines</p>
                </div>
              </div>
              <div class="flex gap-4">
                <div class="text-2xl">🕐</div>
                <div>
                  <p class="text-sm text-gray-400 mb-1">Business Hours</p>
                  <p class="text-white font-semibold">Monday–Friday, 9:00–17:00 (PH Time)</p>
                </div>
              </div>
              <div class="pt-6 border-t border-white/10 flex gap-4">
                <a href="https://facebook.com/voclaria" target="_blank" class="px-4 py-2 rounded-lg bg-blue-600/20 text-blue-300 hover:bg-blue-600/30 transition-all font-semibold text-sm">
                  Facebook
                </a>
                <a href="https://t.me/voclaria" target="_blank" class="px-4 py-2 rounded-lg bg-blue-500/20 text-blue-300 hover:bg-blue-500/30 transition-all font-semibold text-sm">
                  Telegram
                </a>
              </div>
            </div>
          </div>

          <!-- Form -->
          <form @submit="submitInquiry" class="rounded-2xl border border-white/10 bg-gradient-to-br from-white/5 to-white/[0.02] p-10">
            <div v-if="formSuccess" class="mb-6 p-4 rounded-lg bg-green-500/20 border border-green-500/50 text-green-300 text-sm font-bold flex items-center gap-2">
              ✓ Message sent successfully! We'll reply soon.
            </div>
            
            <div class="space-y-5">
              <div>
                <label class="block text-sm font-bold text-gray-300 mb-2" for="inq-name">Full Name</label>
                <input id="inq-name" v-model="nameInput" type="text" required
                       class="w-full rounded-lg bg-white/[0.04] border border-white/10 px-4 py-3 text-white
                              outline-none focus:ring-2 focus:ring-purple-500/80 focus:border-purple-500/40 focus:bg-white/[0.08]
                              placeholder:text-gray-500 transition-all text-base" placeholder="John Doe" />
              </div>

              <div>
                <label class="block text-sm font-bold text-gray-300 mb-2" for="inq-email">Email Address</label>
                <input id="inq-email" v-model="emailInput" type="email" required
                       class="w-full rounded-lg bg-white/[0.04] border border-white/10 px-4 py-3 text-white
                              outline-none focus:ring-2 focus:ring-purple-500/80 focus:border-purple-500/40 focus:bg-white/[0.08]
                              placeholder:text-gray-500 transition-all text-base" placeholder="john@school.edu" />
              </div>

              <div>
                <label class="block text-sm font-bold text-gray-300 mb-2" for="inq-message">Message</label>
                <textarea id="inq-message" v-model="messageInput" rows="5" required
                          class="w-full rounded-lg bg-white/[0.04] border border-white/10 px-4 py-3 text-white
                                 outline-none focus:ring-2 focus:ring-purple-500/80 focus:border-purple-500/40 focus:bg-white/[0.08]
                                 placeholder:text-gray-500 resize-none transition-all text-base" placeholder="Tell us about your school, class size, and goals..."></textarea>
              </div>
            </div>

            <button type="submit"
                    :disabled="submitting || !nameInput || !emailInput || !messageInput"
                    class="mt-6 w-full px-6 py-4 rounded-lg bg-gradient-to-r from-purple-600 to-purple-700 hover:from-purple-500 hover:to-purple-600
                           text-white font-bold transition-all shadow-lg hover:shadow-xl disabled:opacity-50 disabled:cursor-not-allowed transform hover:-translate-y-1 duration-300 text-lg">
              {{ submitting ? 'Sending…' : 'Send Message' }}
            </button>
          </form>
        </div>
      </div>
    </section>

    <!-- FOOTER -->
    <footer class="py-16 border-t border-white/10 bg-[#0B1120]">
      <div class="max-w-7xl mx-auto px-5 sm:px-6 lg:px-8">
        <div class="grid md:grid-cols-3 gap-8 mb-8">
          <div>
            <h4 class="font-bold text-white mb-3">Product</h4>
            <div class="space-y-2">
              <a href="#features" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Features</a><br>
              <a href="#faq" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">FAQ</a>
            </div>
          </div>
          <div>
            <h4 class="font-bold text-white mb-3">Company</h4>
            <div class="space-y-2">
              <a href="#members" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Team</a><br>
              <a href="#inquiries" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Contact</a>
            </div>
          </div>
          <div>
            <h4 class="font-bold text-white mb-3">Follow Us</h4>
            <div class="space-y-2">
              <a href="https://facebook.com/voclaria" target="_blank" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Facebook</a><br>
              <a href="https://t.me/voclaria" target="_blank" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Telegram</a>
            </div>
          </div>
        </div>
        <div class="border-t border-white/10 pt-8 flex flex-col md:flex-row justify-between items-center">
          <p class="text-gray-400 text-sm">© 2025 Voclaria. All rights reserved.</p>
          <div class="flex gap-6 mt-4 md:mt-0">
            <a :href="`mailto:${CONTACT_EMAIL}`" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Privacy</a>
            <a href="#" class="text-gray-400 hover:text-purple-400 transition-colors text-sm">Terms</a>
          </div>
        </div>
      </div>
    </footer>
  </section>
</template>

<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from 'vue'
import logoUrl from '@/assets/Speaksy.png'

const DOWNLOAD_URL = 'https://github.com/Yukiiii2/Voclaria-landing/releases/download/v1.0.0/voclaria-final.apk'
const RELEASE_URL = 'https://github.com/Yukiiii2/Voclaria-landing/releases/tag/v1.0.0'
const APP_SOURCE_URL = 'https://github.com/Yukiiii2/Voclaria'
const CONTACT_EMAIL = 'voclaria.app@gmail.com'

const menuOpen = ref(false)
const menuToggle = ref(null)
const qrDataUrl = ref('')
const qrError = ref('')
const nameInput = ref('')
const emailInput = ref('')
const messageInput = ref('')
const formStatus = ref('')
let revealObserver

const navItems = [
  { label: 'For students', href: '#students' },
  { label: 'For educators', href: '#educators' },
  { label: 'How it works', href: '#how-it-works' },
  { label: 'Team', href: '#team' },
  { label: 'FAQ', href: '#faq' },
  { label: 'Contact', href: '#contact' },
]

const studentFeatures = [
  { glyph: 'Aa', eyebrow: 'Read', title: 'Build reading confidence', description: 'Work through basic and advanced reading modules, comprehension activities, and recorded reading practice.' },
  { glyph: 'Wv', eyebrow: 'Speak', title: 'Practice your delivery', description: 'Use guided speaking lessons, voice recording, and private or live video practice to strengthen communication skills.' },
  { glyph: 'AI', eyebrow: 'Reflect', title: 'Review useful feedback', description: 'Voclaria includes analysis flows for speaking and reading practice so learners can revisit results and improve.' },
  { glyph: '%', eyebrow: 'Progress', title: 'See learning progress', description: 'Student dashboards surface speaking and reading progress, completed modules, and upcoming learning work.' },
]

const educatorFeatures = [
  { number: '01', title: 'Create and manage classes', description: 'Teacher workflows include class creation, editing, module management, and student administration.' },
  { number: '02', title: 'Follow student progress', description: 'Dashboard views combine module completion with speaking and reading progress for enrolled learners.' },
  { number: '03', title: 'Support live practice', description: 'Teachers can run live-session workflows and participate in the application’s learning community.' },
]

const verifiedScope = [
  'Student and teacher account paths',
  'Reading and speaking lesson modules',
  'Voice, camera, and video practice flows',
  'Class, community, and live-session tools',
  'Supabase-backed authentication and data',
  'Progress views for learners and educators',
]

const steps = [
  { number: '01', title: 'Choose your learning path', description: 'Start with reading or speaking, then move through the level and module that fits your current practice.' },
  { number: '02', title: 'Practice in the app', description: 'Complete guided activities using text, audio, voice recording, or video where the lesson calls for it.' },
  { number: '03', title: 'Review and keep moving', description: 'Return to results and progress views, then continue with upcoming modules or teacher-assigned work.' },
]

const team = [
  { initials: 'DF', name: 'Dianna Mae Flores', role: 'Project Manager' },
  { initials: 'XB', name: 'Xaian Paul Belderol', role: 'UI/UX Designer · Frontend Developer' },
  { initials: 'RD', name: 'Rockford Jade Dagohoy', role: 'AI Developer · Backend Developer · System Analyst' },
  { initials: 'EA', name: 'Earl Ang', role: 'UI/UX Designer · Frontend & Backend Developer' },
]

const faqs = [
  { question: 'Where does the Android download come from?', answer: 'The download button and QR code point to the Voclaria v1.0.0 APK hosted in this project’s official GitHub Release. The file is approximately 121.5 MiB.' },
  { question: 'Which platforms are currently available?', answer: 'The current public release is an Android APK. A public iOS download is not listed, so this page does not promise an iOS release or timeline.' },
  { question: 'Why may Android ask for installation permission?', answer: 'Because Voclaria is distributed as a direct APK instead of through an app store, Android may ask you to allow installs from your browser or file manager. Review the Android prompt before continuing.' },
  { question: 'What device permissions can the app use?', answer: 'The application source declares camera, microphone, media, network, and related permissions for recording and live-session features. Android controls when those permissions are requested.' },
  { question: 'Does every feature work offline?', answer: 'Voclaria uses network-backed services for authentication, stored progress, community features, and analysis workflows. A reliable internet connection should be expected for the full experience.' },
  { question: 'Is pricing published?', answer: 'No verified public pricing is currently listed. Use the contact section for availability or school-related questions.' },
]

const buttonHref = computed(() => {
  const url = new URL(DOWNLOAD_URL)
  url.searchParams.set('utm_source', 'landing-page')
  url.searchParams.set('utm_medium', 'download-button')
  return url.toString()
})

function closeMenu() {
  menuOpen.value = false
}

function navigateTo(href, event) {
  event?.preventDefault?.()
  closeMenu()

  const target = document.querySelector(href)
  if (!target) return

  const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  target.scrollIntoView({ behavior: reducedMotion ? 'auto' : 'smooth', block: 'start' })
  window.history.pushState(null, '', href)
}

async function handleEscape(event) {
  if (event.key !== 'Escape' || !menuOpen.value) return

  closeMenu()
  await nextTick()
  menuToggle.value?.focus()
}

function submitInquiry() {
  if (!nameInput.value || !emailInput.value || !messageInput.value) return

  const subject = encodeURIComponent('Voclaria Inquiry')
  const body = encodeURIComponent(`Name: ${nameInput.value}\nEmail: ${emailInput.value}\n\nMessage:\n${messageInput.value}`)
  formStatus.value = 'Your email app should open with a prepared draft. Review it and press Send there to deliver your inquiry.'
  window.location.href = `mailto:${CONTACT_EMAIL}?subject=${subject}&body=${body}`
}

async function setupQrCode() {
  if (!window.matchMedia('(min-width: 1024px)').matches) return

  try {
    const { default: QRCode } = await import('qrcode')
    const url = new URL(DOWNLOAD_URL)
    url.searchParams.set('utm_source', 'qr-code')
    url.searchParams.set('utm_medium', 'landing-page')
    qrDataUrl.value = await QRCode.toDataURL(url.toString(), {
      errorCorrectionLevel: 'M',
      margin: 2,
      width: 420,
      color: { dark: '#11142A', light: '#FFFFFFFF' },
    })
  } catch (error) {
    console.error('Failed generating QR:', error)
    qrError.value = 'QR code unavailable. Use the Android download button instead.'
  }
}

function setupReveals() {
  const elements = document.querySelectorAll('[data-reveal]')
  if (!('IntersectionObserver' in window) || window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    elements.forEach((element) => element.classList.add('is-visible'))
    return
  }

  revealObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return
      entry.target.classList.add('is-visible')
      revealObserver?.unobserve(entry.target)
    })
  }, { threshold: 0.12, rootMargin: '0px 0px -48px' })

  elements.forEach((element) => {
    element.classList.add('reveal-ready')
    revealObserver.observe(element)
  })
}

onMounted(async () => {
  document.addEventListener('keydown', handleEscape)
  await setupQrCode()
  await nextTick()
  setupReveals()
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleEscape)
  revealObserver?.disconnect()
})
</script>

<template>
  <div class="site-shell">
    <a class="skip-link" href="#main-content">Skip to main content</a>

    <header class="site-header">
      <div class="page-width header-inner">
        <a class="brand" href="#top" aria-label="Voclaria home" @click="navigateTo('#top', $event)">
          <span class="brand-mark"><img :src="logoUrl" alt="" /></span>
          <span>Voclaria</span>
        </a>

        <nav class="desktop-nav" aria-label="Primary navigation">
          <a v-for="item in navItems" :key="item.href" :href="item.href" @click="navigateTo(item.href, $event)">{{ item.label }}</a>
        </nav>

        <a class="header-download" :href="buttonHref">Download app</a>

        <button ref="menuToggle" class="menu-toggle" type="button" :aria-expanded="menuOpen" aria-controls="mobile-navigation" :aria-label="menuOpen ? 'Close navigation menu' : 'Open navigation menu'" @click="menuOpen = !menuOpen">
          <span :class="{ open: menuOpen }"></span>
          <span :class="{ open: menuOpen }"></span>
        </button>
      </div>

      <div v-show="menuOpen" id="mobile-navigation" class="mobile-nav-panel">
        <nav class="page-width mobile-nav" aria-label="Mobile navigation">
          <a v-for="item in navItems" :key="item.href" :href="item.href" @click="navigateTo(item.href, $event)">{{ item.label }} <span aria-hidden="true">↗</span></a>
          <a class="mobile-download" :href="buttonHref" @click="closeMenu">Download Android APK</a>
        </nav>
      </div>
    </header>

    <main id="main-content">
      <section id="top" class="hero-section">
        <div class="hero-glow hero-glow-one"></div>
        <div class="hero-glow hero-glow-two"></div>

        <div class="page-width hero-grid">
          <div class="hero-copy">
            <p class="eyebrow hero-eyebrow"><span></span> Learning that finds your voice</p>
            <h1><span class="heading-primary">Read with clarity.</span><em class="heading-accent">Speak with confidence.</em></h1>
            <p class="hero-description">Voclaria brings reading and speaking practice into one focused mobile experience, with guided lessons, recorded activities, feedback flows, and progress you can revisit.</p>

            <div class="hero-actions">
              <a class="button button-primary" :href="buttonHref"><span>Download for Android</span><span aria-hidden="true">↓</span></a>
              <a class="button button-secondary" href="#students" @click="navigateTo('#students', $event)">Explore the experience <span aria-hidden="true">↘</span></a>
            </div>

            <div class="release-line" aria-label="Current release information">
              <span class="status-dot"></span>
              <span><strong>v1.0.0</strong> · 121.5 MiB · Android APK</span>
              <a :href="RELEASE_URL" target="_blank" rel="noopener noreferrer">View release</a>
            </div>
          </div>

          <div class="product-stage" aria-label="Voclaria branded product presentation">
            <div class="orbit orbit-one"></div><div class="orbit orbit-two"></div><div class="phone-shadow"></div>
            <div class="phone-frame">
              <div class="phone-speaker"></div>
              <div class="phone-screen">
                <div class="phone-topline"><img :src="logoUrl" alt="" /><span>VOCLARIA</span><span class="phone-pill">LEARN</span></div>
                <div class="phone-welcome"><small>Your practice space</small><strong>Build your voice,<br />one lesson at a time.</strong></div>
                <div class="mode-card mode-reading"><span class="mode-icon">Aa</span><span><small>READING</small><strong>Guided practice</strong></span><b>↗</b></div>
                <div class="mode-card mode-speaking"><span class="mode-icon">Wv</span><span><small>SPEAKING</small><strong>Voice activities</strong></span><b>↗</b></div>
                <div class="progress-card"><div><small>LEARNING PROGRESS</small><strong>Continue your modules</strong></div><div class="progress-rings" aria-hidden="true"><i></i><i></i><i></i></div></div>
              </div>
            </div>
            <div class="floating-chip chip-reading"><span>Aa</span> Reading modules</div>
            <div class="floating-chip chip-speaking"><span>◉</span> Speaking practice</div>
            <p class="product-caption">Branded product illustration based on implemented app capabilities.</p>
          </div>
        </div>

        <div class="page-width hero-rail" aria-label="Voclaria focus areas"><span>Reading</span><i></i><span>Speaking</span><i></i><span>Practice</span><i></i><span>Progress</span></div>
      </section>

      <section id="students" class="section student-section">
        <div class="page-width">
          <div class="section-heading split-heading" data-reveal>
            <div><p class="eyebrow"><span></span> For students</p><h2><span class="heading-primary">A calmer place to</span><em class="heading-accent">practice out loud.</em></h2></div>
            <p>Build the habits behind clearer reading and speaking through focused modules, repeatable practice, and progress you can return to.</p>
          </div>
          <div class="feature-grid">
            <article v-for="(feature, index) in studentFeatures" :key="feature.title" class="feature-card" data-reveal :style="{ '--delay': `${index * 70}ms` }">
              <div class="feature-glyph">{{ feature.glyph }}</div><p class="card-eyebrow">{{ feature.eyebrow }}</p><h3>{{ feature.title }}</h3><p>{{ feature.description }}</p><span class="card-index">0{{ index + 1 }}</span>
            </article>
          </div>
        </div>
      </section>

      <section class="section proof-section">
        <div class="page-width proof-grid">
          <div class="proof-copy" data-reveal>
            <p class="eyebrow light"><span></span> Verified product scope</p><h2><span class="heading-primary">Built around</span><em class="heading-accent">real learning flows.</em></h2>
            <p>Voclaria brings reading, speaking, class management, and progress tracking into one connected learning experience for students and educators.</p>
            <a :href="APP_SOURCE_URL" target="_blank" rel="noopener noreferrer" class="text-link">Inspect the application source <span aria-hidden="true">↗</span></a>
          </div>
          <ul class="scope-list" data-reveal>
            <li v-for="(item, index) in verifiedScope" :key="item"><span>0{{ index + 1 }}</span><strong>{{ item }}</strong><i aria-hidden="true">✓</i></li>
          </ul>
        </div>
      </section>

      <section id="educators" class="section educator-section">
        <div class="page-width educator-grid">
          <div class="educator-visual" data-reveal aria-hidden="true">
            <div class="dashboard-card">
              <div class="dashboard-header"><div><small>EDUCATOR OVERVIEW</small><strong>Class progress</strong></div><span>•••</span></div>
              <div class="dashboard-metrics"><div><span>Speaking</span><strong>Modules</strong><i style="--value: 72%"></i></div><div><span>Reading</span><strong>Practice</strong><i style="--value: 58%"></i></div></div>
              <div class="student-row"><b>AM</b><span><strong>Learner progress</strong><small>Reading and speaking</small></span><em>Active</em></div>
              <div class="student-row"><b>JL</b><span><strong>Module overview</strong><small>Assigned activities</small></span><em>Review</em></div>
              <div class="student-row"><b>KS</b><span><strong>Class activity</strong><small>Upcoming work</small></span><em>View</em></div>
            </div>
            <p>Conceptual dashboard composition — not an application screenshot.</p>
          </div>
          <div class="educator-copy" data-reveal>
            <p class="eyebrow"><span></span> For educators</p><h2><span class="heading-primary">Structure the practice.</span><em class="heading-accent">Support the progress.</em></h2>
            <p class="section-intro">Voclaria’s teacher workflows connect class management, learner progress, modules, and live activities in one mobile application.</p>
            <div class="educator-list"><article v-for="feature in educatorFeatures" :key="feature.number"><span>{{ feature.number }}</span><div><h3>{{ feature.title }}</h3><p>{{ feature.description }}</p></div></article></div>
          </div>
        </div>
      </section>

      <section id="how-it-works" class="section process-section">
        <div class="page-width">
          <div class="section-heading centered" data-reveal><p class="eyebrow"><span></span> A focused rhythm</p><h2><span class="heading-primary">Practice. Review.</span><em class="heading-accent">Keep growing.</em></h2></div>
          <div class="process-grid">
            <article v-for="(step, index) in steps" :key="step.number" data-reveal :style="{ '--delay': `${index * 80}ms` }"><div class="process-number">{{ step.number }}</div><div class="process-line"><i></i></div><h3>{{ step.title }}</h3><p>{{ step.description }}</p></article>
          </div>
        </div>
      </section>

      <section class="download-section">
        <div class="page-width download-card" data-reveal>
          <div class="download-copy"><p class="eyebrow light"><span></span> Android release</p><h2><span class="heading-primary">Make room for</span><em class="heading-accent">your next practice.</em></h2><p>Download the current Voclaria APK directly from the official GitHub Release.</p><a class="button button-light" :href="buttonHref">Download v1.0.0 <span aria-hidden="true">↓</span></a><small>121.5 MiB · Direct APK installation · Android only</small></div>
          <div class="qr-panel">
            <template v-if="qrDataUrl"><div class="qr-code"><img :src="qrDataUrl" alt="QR code for the Voclaria Android APK download" /></div><div><strong>Scan to download</strong><span>Open with your Android phone</span></div></template>
            <template v-else><img class="qr-logo" :src="logoUrl" alt="Voclaria logo" /><p>{{ qrError || 'Open this page on desktop to display the download QR code.' }}</p></template>
          </div>
        </div>
      </section>

      <section id="team" class="section team-section">
        <div class="page-width">
          <div class="section-heading split-heading" data-reveal><div><p class="eyebrow"><span></span> The team</p><h2><span class="heading-primary">Designed and built</span><em class="heading-accent">with purpose.</em></h2></div><p>Voclaria is the work of a four-person project team spanning product coordination, design, frontend, backend, AI, and systems analysis.</p></div>
          <div class="team-grid"><article v-for="(member, index) in team" :key="member.name" data-reveal :style="{ '--delay': `${index * 65}ms` }"><div class="member-avatar"><span>{{ member.initials }}</span></div><span class="member-number">0{{ index + 1 }}</span><h3>{{ member.name }}</h3><p>{{ member.role }}</p></article></div>
        </div>
      </section>

      <section id="faq" class="section faq-section">
        <div class="page-width faq-grid">
          <div class="faq-heading" data-reveal><p class="eyebrow"><span></span> Good to know</p><h2><span class="heading-primary">Questions,</span><em class="heading-accent">answered plainly.</em></h2><p>Clear information about the current Android release and what to expect.</p></div>
          <div class="faq-list" data-reveal><details v-for="faq in faqs" :key="faq.question"><summary><span>{{ faq.question }}</span><i aria-hidden="true"></i></summary><p>{{ faq.answer }}</p></details></div>
        </div>
      </section>

      <section id="contact" class="section contact-section">
        <div class="page-width contact-grid">
          <div class="contact-copy" data-reveal><p class="eyebrow light"><span></span> Contact Voclaria</p><h2><span class="heading-primary">Have a question?</span><em class="heading-accent">Start with an email.</em></h2><p>Ask about the Android release, the learning experience, or using Voclaria in a class setting.</p><div class="contact-details"><a :href="`mailto:${CONTACT_EMAIL}`"><span>Email</span><strong>{{ CONTACT_EMAIL }}</strong></a><div><span>Based in</span><strong>Davao City, Philippines</strong></div></div></div>
          <form class="contact-form" data-reveal @submit.prevent="submitInquiry">
            <div class="field-row"><label for="inquiry-name">Your name</label><input id="inquiry-name" v-model.trim="nameInput" type="text" autocomplete="name" required placeholder="How should we address you?" /></div>
            <div class="field-row"><label for="inquiry-email">Email address</label><input id="inquiry-email" v-model.trim="emailInput" type="email" autocomplete="email" required placeholder="you@example.com" /></div>
            <div class="field-row"><label for="inquiry-message">How can we help?</label><textarea id="inquiry-message" v-model.trim="messageInput" rows="5" required placeholder="Tell us what you’d like to know."></textarea></div>
            <button class="button button-light form-submit" type="submit">Open email draft <span aria-hidden="true">↗</span></button>
            <p class="form-note">This form opens your default email application. Your message is only delivered after you send it there.</p>
            <p v-if="formStatus" class="form-status" role="status">{{ formStatus }}</p>
          </form>
        </div>
      </section>
    </main>

    <footer class="site-footer">
      <div class="page-width footer-top"><a class="brand footer-brand" href="#top" @click="navigateTo('#top', $event)"><span class="brand-mark"><img :src="logoUrl" alt="" /></span><span>Voclaria</span></a><p>Reading and speaking practice, thoughtfully brought together.</p><a class="footer-download" :href="buttonHref">Download for Android <span aria-hidden="true">↓</span></a></div>
      <div class="page-width footer-bottom"><p>© {{ new Date().getFullYear() }} Voclaria</p><nav aria-label="Footer navigation"><a href="#students" @click="navigateTo('#students', $event)">Students</a><a href="#educators" @click="navigateTo('#educators', $event)">Educators</a><a href="#team" @click="navigateTo('#team', $event)">Team</a><a href="#faq" @click="navigateTo('#faq', $event)">FAQ</a><a href="#contact" @click="navigateTo('#contact', $event)">Contact</a></nav><a :href="RELEASE_URL" target="_blank" rel="noopener noreferrer">GitHub release ↗</a></div>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { marked } from 'marked'
import {
  site, stats, aboutCards, projects, skillGroups, experience, education, certifications,
} from './data/site.js'

// ---------- Theme Management ----------
const theme = ref('light')
function applyTheme(t) {
  theme.value = t
  document.documentElement.setAttribute('data-theme', t)
  try { localStorage.setItem('theme', t) } catch (e) { /* ignore */ }
}
function toggleTheme() {
  applyTheme(theme.value === 'light' ? 'dark' : 'light')
}

// ---------- Navigation & Scrollspy ----------
const menuOpen = ref(false)
const showTop = ref(false)
const activeSection = ref('home')
const links = [
  ['home', 'Home'],
  ['about', 'About'],
  ['projects', 'Projects'],
  ['skills', 'Skills'],
  ['experience', 'Experience'],
  // ['updates', 'Updates'],
  ['contact', 'Contact'],
]

const onScroll = () => {
  showTop.value = window.scrollY > 450

  const sectionIds = ['contact', /* 'updates', */ 'experience', 'skills', 'projects', 'about', 'home']
  for (const id of sectionIds) {
    const el = document.getElementById(id)
    if (el) {
      const rect = el.getBoundingClientRect()
      if (rect.top <= 220) {
        activeSection.value = id
        break
      }
    }
  }
}

// ---------- Photo Fallback ----------
const photoOk = ref(true)
const photoUrl = "/photo.jpg"

// ---------- Projects Filter & Counts ----------
const filters = ['All', 'Support', 'Systems', 'Development', 'Community']
const activeFilter = ref('All')

const filterCounts = computed(() => {
  const counts = { All: projects.length }
  projects.forEach((p) => {
    counts[p.category] = (counts[p.category] || 0) + 1
  })
  return counts
})

const shownProjects = computed(() =>
  activeFilter.value === 'All' ? projects : projects.filter((p) => p.category === activeFilter.value),
)

// ---------- Balanced Skills Columns (prevents awkward blank spaces) ----------
const skillColumns = computed(() => [
  [
    { ...skillGroups[0], icon: 'support' },
    { ...skillGroups[1], icon: 'infra' },
  ],
  [
    { ...skillGroups[2], icon: 'dev' },
    { ...skillGroups[3], icon: 'working' },
  ],
])

// ---------- Email Copy to Clipboard ----------
const emailCopied = ref(false)
async function copyEmail() {
  try {
    await navigator.clipboard.writeText(site.email)
    emailCopied.value = true
    setTimeout(() => { emailCopied.value = false }, 2200)
  } catch (err) {
    // Fallback if clipboard API is restricted
    const input = document.createElement('input')
    input.value = site.email
    document.body.appendChild(input)
    input.select()
    document.execCommand('copy')
    document.body.removeChild(input)
    emailCopied.value = true
    setTimeout(() => { emailCopied.value = false }, 2200)
  }
}

// ---------- Updates (Markdown Posts) ----------
const rawPosts = import.meta.glob('./posts/*.md', { query: '?raw', import: 'default', eager: true })
function parsePost(path, raw) {
  const match = raw.match(/^---\s*([\s\S]*?)\s*---\s*([\s\S]*)$/)
  const meta = {}
  let body = raw
  if (match) {
    match[1].split('\n').forEach((line) => {
      const i = line.indexOf(':')
      if (i > -1) meta[line.slice(0, i).trim()] = line.slice(i + 1).trim()
    })
    body = match[2]
  }
  return {
    slug: path.split('/').pop().replace('.md', ''),
    title: meta.title || 'Untitled',
    date: meta.date || '',
    summary: meta.summary || '',
    html: marked.parse(body),
  }
}

const posts = Object.entries(rawPosts)
  .map(([p, r]) => parsePost(p, r))
  .sort((a, b) => (a.date < b.date ? 1 : -1))

const openPost = ref(null)
const previewProject = ref(null)

const formatDate = (d) =>
  d ? new Date(d).toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' }) : ''

const onKeydown = (e) => {
  if (e.key === 'Escape') {
    if (openPost.value) openPost.value = null
    if (previewProject.value) previewProject.value = null
  }
}

onMounted(() => {
  let saved = null
  try { saved = localStorage.getItem('theme') } catch (e) { /* ignore */ }
  applyTheme(saved || (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'))
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('keydown', onKeydown)
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('keydown', onKeydown)
})

const toTop = () => window.scrollTo({ top: 0, behavior: 'smooth' })
</script>

<template>
  <!-- ==================== HEADER ==================== -->
  <header class="top">
    <div class="wrap top-in">
      <a href="#home" class="brand-link" aria-label="Go to home">
        <span class="brand-logo-badge">{{ site.initials }}</span>
        <span class="brand-text">{{ site.shortName }}<span>.</span></span>
      </a>

      <nav :class="['links', { open: menuOpen }]">
        <a
          v-for="[id, label] in links"
          :key="id"
          :href="'#' + id"
          :class="['nav-item', { active: activeSection === id }]"
          @click="menuOpen = false"
        >
          {{ label }}
        </a>
      </nav>

      <div class="tools">
        <a :href="site.cvFile" download class="nav-cv-btn">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
            <polyline points="7 10 12 15 17 10"></polyline>
            <line x1="12" y1="15" x2="12" y2="3"></line>
          </svg>
          <span>CV</span>
        </a>

        <button
          class="theme-toggle-btn"
          @click="toggleTheme"
          :aria-label="theme === 'light' ? 'Switch to dark mode' : 'Switch to light mode'"
        >
          <!-- Moon icon for light mode -->
          <svg v-if="theme === 'light'" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
          </svg>
          <!-- Sun icon for dark mode -->
          <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="5"></circle>
            <line x1="12" y1="1" x2="12" y2="3"></line>
            <line x1="12" y1="21" x2="12" y2="23"></line>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
            <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
            <line x1="1" y1="12" x2="3" y2="12"></line>
            <line x1="21" y1="12" x2="23" y2="12"></line>
            <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
            <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
          </svg>
        </button>

        <button class="burger" @click="menuOpen = !menuOpen" aria-label="Toggle navigation menu">
          <svg v-if="!menuOpen" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="3" y1="12" x2="21" y2="12"></line>
            <line x1="3" y1="6" x2="21" y2="6"></line>
            <line x1="3" y1="18" x2="21" y2="18"></line>
          </svg>
          <svg v-else width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
    </div>
  </header>

  <main>
    <!-- ==================== HERO SECTION ==================== -->
    <section id="home" class="hero">
      <div class="wrap">
        <div class="hero-grid">
          <div>
            <div class="hero-badge-row">
              <span class="status-pill">
                <span class="pulse-dot"></span>
                Available for IT & Systems Roles
              </span>
            </div>

            <h1 class="hero-title">
              Keeping <span class="gradient-text">systems reliable</span> & people empowered.
            </h1>

            <p class="hero-intro">{{ site.intro }}</p>

            <div class="role-tags">
              <span v-for="b in site.badges" :key="b" class="role-tag">{{ b }}</span>
            </div>

            <div class="cta-row">
              <a href="#projects" class="btn-primary">
                <span>Explore Projects</span>
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="5" y1="12" x2="19" y2="12"></line>
                  <polyline points="12 5 19 12 12 19"></polyline>
                </svg>
              </a>

              <a :href="site.cvFile" download class="btn-secondary">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
                  <polyline points="7 10 12 15 17 10"></polyline>
                  <line x1="12" y1="15" x2="12" y2="3"></line>
                </svg>
                <span>Download CV</span>
              </a>

              <a href="#contact" class="btn-ghost-link">
                <span>Say Hello</span>
                <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M7 17L17 7"></path>
                  <polyline points="7 7 17 7 17 17"></polyline>
                </svg>
              </a>
            </div>
          </div>

          <!-- Avatar Card -->
          <div class="avatar-card-wrapper">
            <div class="avatar-card">
              <div class="avatar-inner">
                <img v-if="photoOk" :src="photoUrl" :alt="site.name" @error="photoOk = false" />
                <div v-else class="avatar-fallback">
                  <span class="avatar-initials">{{ site.initials }}</span>
                </div>
              </div>
              <div class="avatar-badge-bottom">
                <div class="avatar-location">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path>
                    <circle cx="12" cy="10" r="3"></circle>
                  </svg>
                  <span>{{ site.location }}</span>
                </div>
                <span class="avatar-exp-chip">5+ Yrs Exp</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Stats Strip -->
        <div class="stats-strip">
          <div v-for="s in stats" :key="s.label" class="stat-box">
            <span class="stat-val">{{ s.value }}</span>
            <span class="stat-lbl">{{ s.label }}</span>
          </div>
        </div>
      </div>
    </section>

    <!-- ==================== ABOUT SECTION ==================== -->
    <section id="about" class="sec">
      <div class="wrap">
        <div class="sec-header">
          <div class="sec-eyebrow">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="4 17 10 11 4 5"></polyline><line x1="12" y1="19" x2="20" y2="19"></line></svg>
            01 // About Me
          </div>
          <h2 class="sec-title">Driving Reliability & Impact Through Technology</h2>
          <p class="sec-subtitle">Hands-on ICT Officer with 5+ years of end-user support, server management, and full-stack development experience.</p>
        </div>

        <div class="about-grid">
          <div class="about-bio-card">
            <p class="lead-p">
              I am an <strong>ICT Officer</strong> at <strong>Youth Net and Counselling (YONECO)</strong> in Zomba, Malawi. Since October 2021, I have been responsible for keeping organizational systems healthy, resolving hardware and software issues for staff and external users, and administering Microsoft 365, servers, and networks.
            </p>
            <p class="sub-lead-p">
              Beyond infrastructure, I build custom web applications with Laravel, Vue.js, and MySQL that replace paper-based bottlenecks with streamlined digital workflows.
            </p>
            <div class="highlight-box">
              "I also mentor young girls in technology through the <strong>GIRL Technovation Program</strong>. My goal is to inspire more young women to build meaningful careers in ICT."
            </div>
          </div>

          <div class="pillars-grid">
            <div v-for="c in aboutCards" :key="c.title" class="pillar-card">
              <div :class="['pillar-icon-box', 'icon-' + c.icon]">
                <!-- Support icon -->
                <svg v-if="c.icon === 'support'" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
                  <line x1="8" y1="21" x2="16" y2="21"></line>
                  <line x1="12" y1="17" x2="12" y2="21"></line>
                </svg>
                <!-- Code icon -->
                <svg v-else-if="c.icon === 'code'" width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="16 18 22 12 16 6"></polyline>
                  <polyline points="8 6 2 12 8 18"></polyline>
                </svg>
                <!-- People / Mentorship icon -->
                <svg v-else width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
                  <circle cx="9" cy="7" r="4"></circle>
                  <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
                  <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
                </svg>
              </div>
              <div class="pillar-content">
                <h3>{{ c.title }}</h3>
                <p>{{ c.text }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ==================== PROJECTS SECTION ==================== -->
    <section id="projects" class="sec">
      <div class="wrap">
        <div class="sec-header">
          <div class="sec-eyebrow">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polygon points="12 2 2 7 12 12 22 7 12 2"></polygon><polyline points="2 17 12 22 22 17"></polyline><polyline points="2 12 12 17 22 12"></polyline></svg>
            02 // Portfolio & Systems
          </div>
          <h2 class="sec-title">Featured Projects & Systems</h2>
          <p class="sec-subtitle">Explore digital tools, enterprise software solutions, and infrastructure operations I've led or built.</p>
        </div>

        <!-- Filter Tabs with counts -->
        <div class="filter-bar">
          <button
            v-for="f in filters"
            :key="f"
            :class="['filter-pill', { active: activeFilter === f }]"
            @click="activeFilter = f"
          >
            <span>{{ f }}</span>
            <span class="filter-count">{{ filterCounts[f] || 0 }}</span>
          </button>
        </div>

        <!-- Projects Card Grid -->
        <div class="projects-grid">
          <article v-for="p in shownProjects" :key="p.title" class="project-card">
            <!-- Optional Project Picture with Lightbox Preview -->
            <div
              v-if="p.image && !p.imgFailed"
              class="project-img-wrapper"
              @click="previewProject = p"
              role="button"
              tabindex="0"
              :aria-label="'Preview image for ' + p.title"
              @keydown.enter="previewProject = p"
            >
              <img
                :src="p.image"
                :alt="p.title"
                class="project-img"
                loading="lazy"
                @error="p.imgFailed = true"
              />
              <div class="project-img-zoom-hint">
                <svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                  <circle cx="11" cy="11" r="8"></circle>
                  <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                  <line x1="11" y1="8" x2="11" y2="14"></line>
                  <line x1="8" y1="11" x2="14" y2="11"></line>
                </svg>
                <span>Click to view</span>
              </div>
            </div>

            <div class="project-body">
              <div class="project-top">
                <span :class="['cat-badge', 'cat-' + p.category.toLowerCase()]">
                  {{ p.category }}
                </span>
                <a
                  v-if="p.link"
                  :href="p.link"
                  target="_blank"
                  rel="noopener"
                  class="project-link-icon"
                  :aria-label="'View ' + p.title"
                >
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path>
                    <polyline points="15 3 21 3 21 9"></polyline>
                    <line x1="10" y1="14" x2="21" y2="3"></line>
                  </svg>
                </a>
              </div>
              <h3 class="project-title">{{ p.title }}</h3>
              <p class="project-desc">{{ p.text }}</p>
            </div>

            <div class="project-tags">
              <span v-for="t in p.tags" :key="t" class="tech-tag">{{ t }}</span>
            </div>
          </article>
        </div>
      </div>
    </section>

    <!-- ==================== SKILLS SECTION ==================== -->
    <section id="skills" class="sec">
      <div class="wrap">
        <div class="sec-header">
          <div class="sec-eyebrow">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>
            03 // Capabilities
          </div>
          <h2 class="sec-title">Technical Skills & Tooling</h2>
          <p class="sec-subtitle">Comprehensive toolkit across systems administration, cloud, development, and team collaboration.</p>
        </div>

        <div class="skills-columns">
          <div v-for="(col, colIdx) in skillColumns" :key="colIdx" class="skills-col">
            <div v-for="g in col" :key="g.title" class="skill-card">
              <div class="skill-card-head">
                <div :class="['skill-icon-wrap', 'icon-' + g.icon]">
                  <!-- Support icon -->
                  <svg v-if="g.icon === 'support'" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
                    <line x1="8" y1="21" x2="16" y2="21"></line>
                    <line x1="12" y1="17" x2="12" y2="21"></line>
                  </svg>
                  <!-- Networks / Infrastructure icon -->
                  <svg v-else-if="g.icon === 'infra'" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <rect x="2" y="2" width="20" height="8" rx="2" ry="2"></rect>
                    <rect x="2" y="14" width="20" height="8" rx="2" ry="2"></rect>
                    <line x1="6" y1="6" x2="6.01" y2="6"></line>
                    <line x1="6" y1="18" x2="6.01" y2="18"></line>
                  </svg>
                  <!-- Development icon -->
                  <svg v-else-if="g.icon === 'dev'" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <polyline points="16 18 22 12 16 6"></polyline>
                    <polyline points="8 6 2 12 8 18"></polyline>
                  </svg>
                  <!-- Working Skills icon -->
                  <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
                    <circle cx="9" cy="7" r="4"></circle>
                    <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
                    <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
                  </svg>
                </div>
                <div class="skill-head-info">
                  <h3>{{ g.title }}</h3>
                  <span class="skill-count">{{ g.items.length }} skills</span>
                </div>
              </div>
              <div class="skill-chips">
                <span v-for="item in g.items" :key="item" class="skill-chip">
                  {{ item }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ==================== EXPERIENCE & EDUCATION SECTION ==================== -->
    <section id="experience" class="sec">
      <div class="wrap">
        <div class="sec-header">
          <div class="sec-eyebrow">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><rect x="2" y="7" width="20" height="14" rx="2" ry="2"></rect><path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16"></path></svg>
            04 // Experience & Credentials
          </div>
          <h2 class="sec-title">Career Track & Education</h2>
          <p class="sec-subtitle">My professional journey in IT operations and my academic qualifications.</p>
        </div>

        <div class="exp-edu-layout">
          <!-- Timeline Column -->
          <div class="timeline">
            <div v-for="e in experience" :key="e.title + e.dates" class="timeline-item">
              <div class="timeline-dot">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="20 6 9 17 4 12"></polyline>
                </svg>
              </div>
              <div class="timeline-card">
                <div class="timeline-header">
                  <h3 class="timeline-role">{{ e.title }}</h3>
                  <span class="timeline-date-chip">{{ e.dates }}</span>
                </div>
                <div class="timeline-org">
                  <span>{{ e.org }}</span>
                  <span class="timeline-org-place">· {{ e.place }}</span>
                </div>
                <ul class="timeline-list">
                  <li v-for="pt in e.points" :key="pt">
                    <span class="timeline-bullet">▸</span>
                    <span>{{ pt }}</span>
                  </li>
                </ul>
              </div>
            </div>
          </div>

          <!-- Education & Certifications Column -->
          <div class="side-edu-column">
            <!-- Education -->
            <div class="edu-card">
              <h3 class="side-card-title">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M22 10v6M2 10l10-5 10 5-10 5z"></path>
                  <path d="M6 12v5c3 3 9 3 12 0v-5"></path>
                </svg>
                Education
              </h3>
              <div v-for="ed in education" :key="ed.title" class="edu-item">
                <h4 class="edu-degree">{{ ed.title }}</h4>
                <div class="edu-org">{{ ed.org }}</div>
                <div class="edu-meta">
                  <span class="edu-year">{{ ed.dates }}</span>
                  <span class="edu-badge">{{ ed.note }}</span>
                </div>
              </div>
            </div>

            <!-- Certifications -->
            <div class="edu-card">
              <h3 class="side-card-title">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <circle cx="12" cy="8" r="7"></circle>
                  <polyline points="8.21 13.89 7 23 12 20 17 23 15.79 13.88"></polyline>
                </svg>
                Certifications
              </h3>
              <div class="certs-list">
                <div v-for="c in certifications" :key="c" class="cert-item">
                  <span class="cert-icon">
                    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                      <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path>
                      <polyline points="22 4 12 14.01 9 11.01"></polyline>
                    </svg>
                  </span>
                  <span>{{ c }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ==================== UPDATES SECTION ==================== -->
    <!--
    <section id="updates" class="sec">
      <div class="wrap">
        <div class="sec-header">
          <div class="sec-eyebrow">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
            05 // Insights & Updates
          </div>
          <h2 class="sec-title">Tech Notes & Learnings</h2>
          <p class="sec-subtitle">Articles on technical troubleshooting, software architecture, and technology mentorship.</p>
        </div>

        <div v-if="posts.length" class="posts-grid">
          <button v-for="p in posts" :key="p.slug" class="post-card" @click="openPost = p">
            <div class="post-meta-row">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
                <line x1="16" y1="2" x2="16" y2="6"></line>
                <line x1="8" y1="2" x2="8" y2="6"></line>
                <line x1="3" y1="10" x2="21" y2="10"></line>
              </svg>
              <span>{{ formatDate(p.date) }}</span>
            </div>
            <h3 class="post-title">{{ p.title }}</h3>
            <p class="post-summary">{{ p.summary }}</p>
            <span class="post-read-link">
              Read article
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <line x1="5" y1="12" x2="19" y2="12"></line>
                <polyline points="12 5 19 12 12 19"></polyline>
              </svg>
            </span>
          </button>
        </div>
        <p v-else class="post-summary">No updates published yet.</p>
      </div>
    </section>
    -->

    <!-- ==================== CONTACT SECTION ==================== -->
    <section id="contact" class="sec">
      <div class="wrap">
        <div class="sec-header">
          <div class="sec-eyebrow">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path><polyline points="22,6 12,13 2,6"></polyline></svg>
            05 // Connect
          </div>
          <h2 class="sec-title">Let's Work Together</h2>
          <p class="sec-subtitle">I'm open to IT support, technical support, systems administrator roles, and relocation.</p>
        </div>

        <div class="contact-card-box">
          <div>
            <h3 class="contact-heading">Have an opportunity or question?</h3>
            <p class="contact-p">
              Whether you need dependable technical support, server administration, or full-stack software development, my inbox is always open.
            </p>

            <div class="email-box-copy">
              <a :href="'mailto:' + site.email" class="email-link">{{ site.email }}</a>
              <button
                class="copy-btn"
                :class="{ copied: emailCopied }"
                @click="copyEmail"
                aria-label="Copy email address"
              >
                <svg v-if="!emailCopied" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
                  <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
                </svg>
                <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
                  <polyline points="20 6 9 17 4 12"></polyline>
                </svg>
                <span>{{ emailCopied ? 'Copied!' : 'Copy' }}</span>
              </button>
            </div>
          </div>

          <div class="contact-methods-grid">
            <a :href="site.linkedin" target="_blank" rel="noopener" class="contact-method-item">
              <div class="contact-icon-box">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"></path>
                  <rect x="2" y="9" width="4" height="12"></rect>
                  <circle cx="4" cy="4" r="2"></circle>
                </svg>
              </div>
              <div>
                <div style="font-size: 0.78rem; color: var(--text-dim); text-transform: uppercase;">LinkedIn</div>
                <span>Connect on LinkedIn</span>
              </div>
            </a>

            <a v-if="site.github" :href="site.github" target="_blank" rel="noopener" class="contact-method-item">
              <div class="contact-icon-box">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path>
                </svg>
              </div>
              <div>
                <div style="font-size: 0.78rem; color: var(--text-dim); text-transform: uppercase;">GitHub</div>
                <span>{{ site.github.replace('https://github.com/', '') }}</span>
              </div>
            </a>

            <div class="contact-method-item">
              <div class="contact-icon-box">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path>
                  <circle cx="12" cy="10" r="3"></circle>
                </svg>
              </div>
              <div>
                <div style="font-size: 0.78rem; color: var(--text-dim); text-transform: uppercase;">Based in</div>
                <span>{{ site.location }}</span>
              </div>
            </div>

            <a :href="site.cvFile" download class="contact-method-item">
              <div class="contact-icon-box">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
                  <polyline points="7 10 12 15 17 10"></polyline>
                  <line x1="12" y1="15" x2="12" y2="3"></line>
                </svg>
              </div>
              <div>
                <div style="font-size: 0.78rem; color: var(--text-dim); text-transform: uppercase;">Curriculum Vitae</div>
                <span>Download Resume (DOCX)</span>
              </div>
            </a>
          </div>
        </div>
      </div>
    </section>
  </main>

  <!-- ==================== FOOTER ==================== -->
  <footer>
    <div class="wrap footer-content">
      <div>&copy; {{ new Date().getFullYear() }} {{ site.name }}. All rights reserved.</div>
      <div class="footer-tech-tag">
        <span>Designed with Vue 3 & Vite</span>
      </div>
    </div>
  </footer>

  <!-- ==================== BACK TO TOP ==================== -->
  <button :class="['up', { show: showTop }]" @click="toTop" aria-label="Scroll back to top">
    <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
      <path d="M12 19V5M5 12l7-7 7 7"/>
    </svg>
  </button>

  <!-- ==================== ARTICLE MODAL VIEWER ==================== -->
  <div v-if="openPost" class="veil" @click.self="openPost = null">
    <div class="sheet" role="dialog" aria-modal="true">
      <div class="sheet-head">
        <time class="sheet-date">{{ formatDate(openPost.date) }}</time>
        <button class="shut" @click="openPost = null" aria-label="Close dialog">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
      <h2 class="sheet-title">{{ openPost.title }}</h2>
      <div class="body" v-html="openPost.html"></div>
    </div>
  </div>

  <!-- ==================== IMAGE LIGHTBOX MODAL ==================== -->
  <div v-if="previewProject" class="veil" @click.self="previewProject = null">
    <div class="lightbox-card" role="dialog" aria-modal="true">
      <div class="lightbox-head">
        <div>
          <span :class="['cat-badge', 'cat-' + previewProject.category.toLowerCase()]">
            {{ previewProject.category }}
          </span>
          <h3 class="lightbox-title">{{ previewProject.title }}</h3>
        </div>
        <button class="shut" @click="previewProject = null" aria-label="Close image preview">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
      <div class="lightbox-body">
        <img :src="previewProject.image" :alt="previewProject.title" class="lightbox-img" />
      </div>
    </div>
  </div>
</template>

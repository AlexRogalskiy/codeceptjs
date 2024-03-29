<template>
  <div
    class="theme-container sectionLayout"
    :class="pageClasses"
    @touchstart="onTouchStart"
    @touchend="onTouchEnd"
  >
    <Navbar
      v-if="shouldShowNavbar"
      @toggle-sidebar="toggleSidebar"
    />

    <div class="hero"></div>


    <div class="container post">
      <article class="content">
        <Content ></Content>
      </article>
    </div>

    <div class="sidebar" v-if="$frontmatter.sidebar">
      <Content slot-key="sidebar"></Content>
    </div>

    <Sidebar
      :items="sidebarItems"
      @toggle-sidebar="toggleSidebar"
    >
      <slot
        name="sidebar-top"
        #top
      />
      <slot
        name="sidebar-bottom"
        #bottom
      />
    </Sidebar>

    <div
      class="sidebar-mask"
      @click="toggleSidebar(false)"
    ></div>

    <Footer />
  </div>
</template>

<style lang="stylus">
.sectionLayout .content
  img
    width: 100%
    @apply shadow
  li
    word-break break-all

</style>

<style lang="scss" scoped>
.theme-container {
  @apply bg-gray-100;
}

.sidebar {
  position: fixed;
  top: 80px;
  left: auto;
  bottom: 0;
  right: 0;
  @apply rounded bg-gray-200;
  div {
    @apply px-8 py-4;
  }
}

.hero {
  height: 400px;
  border-bottom: 5px dashed;
  @apply bg-purple-200 border-gray-100;
  background: url(/img/back.png) #805ad5;
}

.container {
  .content {
    background: rgba(255,255,255,0.9);
    margin-top: -405px;
    z-index: 100;
    @apply p-8 rounded-lg;
  }


  max-width: 860px;
  margin: 0 auto;
  @apply mt-20 mb-10;

  img {
    width: 100%;
    @apply shadow;
  }

}


@media (max-width: 600px) {
  .hero {
    display: none;
  }

  .container {
    @apply px-0 text-sm;
    width: 100%;
    padding-left: 0 !important;
    padding-right: 0 !important;
    .content {
      @apply px-4;
      margin-top: 0;
    }
  }
}

@media (max-width: 1100px) {
  .hero {
    display: none;
  }
  .container {
    @apply px-4;
    .content {
      margin-top: 0px;
    }
  }
}

@media (max-width: 1550px) {
  .sidebar {
    top: 0;
    left: 0;
    transform: translateX(-100%);
  }
}

@media (max-width: 1350px) {
  .container {
    @apply px-4;
  }
}
</style>

<script>
import Home from '@theme/components/Home.vue'
import Navbar from '@theme/components/Navbar.vue'
import Page from '@theme/components/Page.vue'
import Sidebar from '@theme/components/Sidebar.vue'
import Footer from '../components/Footer'
import { resolveSidebarItems } from '../util'

export default {
  components: { Home, Page, Sidebar, Navbar, Footer },

  data () {
    return {
      isSidebarOpen: false
    }
  },

  computed: {
    shouldShowNavbar () {
      const { themeConfig } = this.$site
      const { frontmatter } = this.$page
      if (
        frontmatter.navbar === false
        || themeConfig.navbar === false) {
        return false
      }
      return (
        this.$title
        || themeConfig.logo
        || themeConfig.repo
        || themeConfig.nav
        || this.$themeLocaleConfig.nav
      )
    },

    shouldShowSidebar () {
      const { frontmatter } = this.$page
      return (
        !frontmatter.home
        && frontmatter.sidebar !== false
        && this.sidebarItems.length
      )
    },

    sidebarItems () {
      return resolveSidebarItems(
        this.$page,
        this.$page.regularPath,
        this.$site,
        this.$localePath
      )
    },

    pageClasses () {
      const userPageClass = this.$page.frontmatter.pageClass
      return [
        {
          'no-navbar': !this.shouldShowNavbar,
          'sidebar-open': this.isSidebarOpen,
          'no-sidebar': !this.shouldShowSidebar
        },
        userPageClass
      ]
    }
  },

  mounted () {
    this.$router.afterEach(() => {
      this.isSidebarOpen = false
    })
  },

  methods: {
    toggleSidebar (to) {
      this.isSidebarOpen = typeof to === 'boolean' ? to : !this.isSidebarOpen
      this.$emit('toggle-sidebar', this.isSidebarOpen)
    },

    // side swipe
    onTouchStart (e) {
      this.touchStart = {
        x: e.changedTouches[0].clientX,
        y: e.changedTouches[0].clientY
      }
    },

    onTouchEnd (e) {
      const dx = e.changedTouches[0].clientX - this.touchStart.x
      const dy = e.changedTouches[0].clientY - this.touchStart.y
      if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
        if (dx > 0 && this.touchStart.x <= 80) {
          this.toggleSidebar(true)
        } else {
          this.toggleSidebar(false)
        }
      }
    }
  }
}
</script>

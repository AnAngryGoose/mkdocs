# Homepage

[Github :simple-github: ](https://github.com/gethomepage/homepage)

---

A modern, fully static, fast, secure fully proxied, highly customizable application dashboard with integrations for over 100 services and translations into multiple languages. Easily configured via YAML files or through docker label discovery. 

## Features

With features like quick search, bookmarks, weather support, a wide range of integrations and widgets, an elegant and modern design, and a focus on performance, Homepage is your ideal start to the day and a handy companion throughout it.

    Fast - The site is statically generated at build time for instant load times.
    Secure - All API requests to backend services are proxied, keeping your API keys hidden. Constantly reviewed for security by the community.
    For Everyone - Images built for AMD64, ARM64.
    Full i18n - Support for over 40 languages.
    Service & Web Bookmarks - Add custom links to the homepage.
    Docker Integration - Container status and stats. Automatic service discovery via labels.
    Service Integration - Over 100 service integrations, including popular starr and self-hosted apps.
    Information & Utility Widgets - Weather, time, date, search, and more.
    And much more...

## Running Homepage via docker

```yaml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    environment:
      HOMEPAGE_ALLOWED_HOSTS: gethomepage.dev # required, may need port. See gethomepage.dev/installation/#homepage_allowed_hosts
      PUID: 1000 # optional, your user id
      PGID: 1000 # optional, your group id
    ports:
      - 3000:3000
    volumes:
      - /path/to/config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock:ro # optional, for docker integrations
    restart: unless-stopped
```

## Configuration 

Homepage can integrate **ALOT**. 

Refer to the [homepage documentation website](https://gethomepage.dev/) for more information. It has everything.

### Basic config file usage is:

`services.yaml`
    
This will actually define the main app blocks on the page. This is where you define the URL, container, icons, and widget info. 

`settings.yaml`

This is the main way to layout the page. Rows, columns, sections, title, weather, etc.

`widgets.yaml`

This is the non-app/container widgets. Search , glances monitor, etc.

`docker.yaml`

Defines your docker socket for integration - same for proxmox, kube, etc.

`bookmarks.yaml`

Smaller bookmark tabs. Youtube, github, whatever you want. No widgets. 

`custom.css`

Makes your page fancy. I hate CSS but this is mine, there is probably a much better way to do it but oh well. Gruvbox dark theme:

```css
/*==================================
  HOMEPAGE CSS - GRUVBOX DARK 
==================================*/

/*==================================
  1. FONT IMPORTS
==================================*/
@import url('https://fonts.googleapis.com/css2?family=Fira+Mono:wght@400;500;700&display=swap');

/*==================================
  2. CSS VARIABLES
==================================*/
:root {
  --my-font: "Fira Mono", monospace;
  
  /* Gruvbox Dark Palette */
  --gb-bg:       #1d2021;
  --gb-card:     #32302f;
  --gb-cream:    #ebdbb2; /* Main Text */
  
  /* Prism Accents */
  --gb-yellow:   #fabd2f; /* Titles/Borders */
  --gb-green:    #98971a; /* Headers */
  --gb-purple:   #d3869b; /* Hovers */
  --gb-blue:     #83a598; /* Meta/Inputs */
  --gb-orange:   #fe8019; /* Icons */
  --gb-red:      #fb4934; /* Alerts */
  
  /* Settings */
  --border-width: 2px;
  --border-radius: 6px; 
}

/*==================================
  3. BASE STYLES
==================================*/
/* Apply background and font to the document root */
body, html, #__next {
  background-color: var(--gb-bg);
  color: var(--gb-cream); /* Default text is Cream*/
  font-family: var(--my-font);
  -webkit-font-smoothing: antialiased;
}

/* Headers / Group Titles */
h1, h2, h3, .group-title {
  color: var(--gb-green);
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
}

/*==================================
  4. CARDS (Services & Bookmarks)
==================================*/
/* Target the cards specifically */
.services-item, 
.bookmark-item, 
.widget {
  background-color: var(--gb-card);
  border: var(--border-width) solid var(--gb-yellow);
  border-radius: var(--border-radius);
  box-shadow: 4px 4px 0px #1d2021; /* Hard shadow for "retro" feel */
  transition: all 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  color: var(--gb-cream);
}

/* Card Titles - Force Yellow */
.services-item .text-base,
.bookmark-item .text-base,
.service-name {
  color: var(--gb-yellow);
  font-weight: 700;
}

/* Card Descriptions - Force Cream/Blue */
.services-item .text-xs, 
.service-description {
  color: var(--gb-cream);
  opacity: 0.8;
}

/* Hover Effects */
.services-item:hover, 
.bookmark-item:hover,
.widget:hover {
  border-color: var(--gb-purple);
  transform: translate(-2px, -2px);
  box-shadow: 6px 6px 0px var(--gb-purple);
  z-index: 10;
}

/* Remove internal transparency/backgrounds */
.services-item a, 
.services-item div, 
.widget div {
   background: transparent;
}

/*==================================
  5. UI COMPONENTS
==================================*/
/* Icons */
.service-icon img, 
.service-icon svg,
.bookmark-icon img,
.bookmark-icon svg {
   /* filter: sepia(100%) hue-rotate(-30deg) saturate(300%); Trick to tint images orange */
   color: var(--gb-cream);
}

/* Search Bar */
input {
  background-color: var(--gb-card);
  border: 2px solid #3c3836;
  color: var(--gb-cream);
  font-family: var(--my-font);
  border-radius: 20px;
}

input:focus {
  outline: none;
  border-color: var(--gb-orange);
}

input::placeholder {
  color: var(--gb-blue);
  opacity: 0.5;
}


/*top bar text color*/
  .dark\:text-theme-200:is(.dark *), .dark\:text-theme-200\/50:is(.dark *) {
    color: var(--gb-cream);
  }

/* contatier stat text */
.text-center,
span[class*="all"] { color: var(--gb-cream) !important; }

/* ICONS AT TOP BAR */
svg.w-5.h-5 {
   color: var(--gb-yellow);
}

/*==================================
  7. UTILITIES / FOOTER
==================================*/
footer, .settings-toggle {
   display: none;
}

@media screen and (max-width: 768px) {
  #myTab {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 10px;
  }
}
```





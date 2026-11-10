# Voclaria Landing

The official landing page for Voclaria, an educational mobile application focused on guided reading, speaking practice, learning feedback, and progress tracking for students and educators.

Voclaria Landing V2 uses a student-first presentation while retaining dedicated educator workflows, the Android release download, QR code access, team information, FAQs, and the email-based contact flow.

## Highlights

- Responsive student and educator product overviews
- Verified descriptions of reading, speaking, class, and progress workflows
- Direct Android APK download from the official GitHub Release
- Desktop QR code generated from the same release URL
- Keyboard-accessible navigation, FAQ controls, and contact form
- Lightweight reveal animations with `prefers-reduced-motion` support
- Responsive layouts designed for mobile, tablet, and desktop
- Open Graph and Twitter sharing metadata

## Technology

- Vue 3
- Vite
- Tailwind CSS 4 and custom CSS
- `qrcode` for the desktop download QR code

## Requirements

- Node.js 20 or newer
- npm
- Git LFS when cloning the repository with tracked APK assets

## Local development

Install dependencies:

```sh
npm install
```

Start the Vite development server:

```sh
npm run dev
```

Create a production build:

```sh
npm run build
```

The optimized output is written to `dist/`, which is intentionally excluded from Git.

## Android download

The landing page download buttons and desktop QR code point to the official Voclaria v1.0.0 GitHub Release:

- [Download the Voclaria Android APK](https://github.com/Yukiiii2/Voclaria-landing/releases/download/v1.0.0/voclaria-final.apk)
- [View the v1.0.0 release](https://github.com/Yukiiii2/Voclaria-landing/releases/tag/v1.0.0)

The APK is approximately 121.5 MiB. Android may ask the user to allow installation from their browser or file manager because the application is distributed directly rather than through an app store.

## Contact workflow

The contact form validates the visitor's name, email address, and message, then opens a prepared draft in the visitor's default email application. It does not send the message automatically; the visitor must review and send the draft from their email client.

## Testing and quality checks

The project currently provides the Vite production build as its automated compilation check:

```sh
npm run build
npm audit
```

No automated unit-test script is currently configured. Responsive and interaction checks should be performed in a browser before publishing interface changes.

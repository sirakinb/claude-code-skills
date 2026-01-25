# Claude Code Skills

A collection of custom skills for Claude Code.

## Skills

| Skill | Description |
|-------|-------------|
| **build-feature** | Autonomous task loop that picks ready tasks, implements them, updates progress.txt, commits, and repeats |
| **compound-engineering** | Compound Engineering workflow following the Plan → Work → Review → Compound loop |
| **dev-browser** | Browser automation with persistent page state for testing and web interactions |
| **email-best-practices** | Guidance for building deliverable, compliant emails - SPF/DKIM/DMARC, CAN-SPAM, GDPR, webhooks, retry logic |
| **pdf** | Comprehensive PDF manipulation toolkit for extracting, creating, and processing PDFs |
| **prd** | Generate Product Requirements Documents (PRDs) for new features |
| **ralph** | Set up Ralph for autonomous feature development with task dependencies |
| **remotion-best-practices** | Best practices for Remotion - Video creation in React |
| **revenuecat-customer** | Create customers in RevenueCat using Firebase UID for promotional/free access |
| **stripe-best-practices** | Best practices for building Stripe integrations |
| **threejs-animation** | Three.js keyframe animation, skeletal animation, morph targets, animation mixing |
| **threejs-fundamentals** | Three.js scene setup, cameras, renderer, Object3D hierarchy, coordinate systems |
| **threejs-geometry** | Three.js geometry creation - built-in shapes, BufferGeometry, custom geometry, instancing |
| **threejs-interaction** | Three.js interaction - raycasting, controls, mouse/touch input, object selection |
| **threejs-lighting** | Three.js lighting - light types, shadows, environment lighting |
| **threejs-loaders** | Three.js asset loading - GLTF, textures, images, models, async patterns |
| **threejs-materials** | Three.js materials - PBR, basic, phong, shader materials, material properties |
| **threejs-postprocessing** | Three.js post-processing - EffectComposer, bloom, DOF, screen effects |
| **threejs-shaders** | Three.js shaders - GLSL, ShaderMaterial, uniforms, custom effects |
| **threejs-textures** | Three.js textures - texture types, UV mapping, environment maps, texture settings |
| **upgrade-stripe** | Guide for upgrading Stripe API versions and SDKs |
| **vercel-react-best-practices** | React and Next.js performance optimization guidelines from Vercel Engineering |
| **web-design-guidelines** | Review UI code for Web Interface Guidelines compliance and accessibility |

## Installation

To use these skills with Claude Code, copy the skill folders to your Claude Code skills directory:

```bash
cp -r <skill-name> ~/.claude/skills/
```

## Usage

Each skill can be invoked by its trigger phrases. For example:
- `/pdf` - Work with PDF files
- `/prd` - Create a product requirements document
- `/ralph` - Set up autonomous feature development
- "build feature" or "run the loop" - Start the build feature loop
- "plan this feature" or "compound learnings" - Use compound engineering workflow
- "go to [url]" or "test the website" - Use browser automation
- "review my UI" or "check accessibility" - Use web design guidelines
- Building email features or fixing deliverability - Use email best practices
- Working with Three.js - Use the relevant threejs-* skill
- Stripe integration or upgrades - Use stripe-best-practices or upgrade-stripe
- React/Next.js performance - Use vercel-react-best-practices

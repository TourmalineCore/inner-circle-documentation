# PWA Manifest Unification

## Current State:

Inner Circle consists of multiple standalone React/Vite services, each with its own manifest.json configured with *"display": "standalone"*. While individual services can be installed as PWAs, navigation between services breaks the PWA experience, causing browser controls to reappear.

Mobile devices (testes on iPhone 16, iPhone 12, Xiaomi 15) show inconsistent PWA behavior because of different browser implementations, operating system peculiarities, and scope management issues.

## Reason:
The reason is that when navigating each service is treated as a separate PWA. The PWA display mode (standalone) is tied to the ```scope: "/"```  defined in the manifest, and cross-service navigation exits this scope.
Leaving only one manifest file in e.g. *layout-ui* or *employees-ui* has no effect, the services still are treated separately. Manifest file is applied only in the service which you extract as an App.

## Device-Specific Behavior: 

### iPhone 16 (Safari)
  - can install individual services as PWA
  - only the installed service hides browser controls
  - navigation to other services shows browser UI
  - "minimal-ui" mode rather than true "standalone"
### iPhone 12 (Safari)
  - inconsistent PWA installation
  - often opens in browser despite manifest
  - older iOS versions have limited PWA support
### Xiaomi 15 (MIUI Browser?)
  - custom browser with non-standard PWA implementation
  - may not respect manifest settings

## Routing Consideration

1. Ingress-Level Configuration 
- configure ingress to route all traffic through a single domain
- place unified *manifest.json* at root level

Example Configuration:
  URL Structure:
    https://innercircle.com/          # Root with unified manifest
    https://innercircle.com/documents # Documents service
    https://innercircle.com/employees # Employees service
    https://innercircle.com/auth      # Auth service

2. Next.js as shell app
Next.js provides routing capabilities and could potentially serve as the shell/container app. Needs configuration to handle cross-service navigation while maintaining PWA context.

3. Service Worker
- root-level Service Worker proxies all requests
- serve unified manifest from root
- handle cross-service navigation within Service Worker